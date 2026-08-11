# EV-energy-management-gitops

ArgoCD가 감시하는 배포 매니페스트 저장소. 앱 코드는 각자 저장소에 있고, "무엇을 어떤 이미지로,
어떻게 배포할지"는 전부 여기서 선언적으로 관리한다.

## 구조

```
apps/
├─ frontend-eks/      # 사용자 웹(React) — 외부 ingress.yaml + 내부 ingress-internal.yaml
├─ backend-eks/        # Spring Boot API — ingress.yaml/ingress-internal.yaml + servicemonitor.yaml(Prometheus) + hpa-custom-metric.yaml
├─ fastapi-eks/        # ev_ai_inference_api + 재생 워커 — ingress.yaml/ingress-internal.yaml + keda-thermal-runaway.yaml
├─ rul-diagnosis/      # 배터리 진단/매도 제안서 — keda-battery-diagnosis.yaml
├─ charging-demand/    # 충전 수요 예측
└─ frontend/backend/fastapi/  # 구 k3s 배포 흔적 (더 이상 안 씀, EKS로 완전히 이전됨)
```

## 배포 흐름

1. 각 서비스 저장소의 GitHub Actions가 이미지를 빌드해 Docker Hub에 push
2. 같은 워크플로우가 `apps/<서비스명>/deployment.yaml`의 이미지 태그를 새 태그로 갱신해 이 저장소에 커밋
3. ArgoCD가 변경을 감지해 클러스터에 자동 반영 (RollingUpdate, 무중단 배포)

## 네트워크 구조 (2026-08-09 기준)

- **외부(사용자) ALB** — `mijungev.kro.kr` / `www.mijungev.kro.kr`, WAFv2(관리형 규칙 4종 + 레이트리밋)
  적용. frontend/backend/fastapi 세 서비스가 하나의 ALB(`group.name: ev-mgmt`)를 공유.
- **내부(관리자/관제) ALB** — `admin.mijungev.kro.kr`, scheme: internal, 인터넷 경로 없음.
  AWS Client VPN(mutual TLS)으로 접속한 사람만 보안그룹으로 허용. 같은 앱 코드를 재사용하며
  `group.name: ev-mgmt-internal`로 별도 ALB 구성. DNS는 Route 53 프라이빗 호스팅 영역(VPC 전용)으로
  등록해 외부에서는 존재 자체를 알 수 없음.
- 두 ALB가 같은 백엔드 서비스(Pod)를 가리키는 경우가 많아, 서비스별 `alb.ingress.kubernetes.io/healthcheck-path`
  를 명시적으로 고정해둬야 한다(안 하면 readinessProbe 자동유추가 ingress 2개일 때 깨짐 — 에러 해결
  기록 참고).

## 오토스케일링 / 관측

- **KEDA + SQS**: `rul-diagnosis`(배터리진단), `fastapi-eks`(열폭주 감지) — 큐 길이 기준 1~6대
- **HPA custom metrics**: `backend-eks` — Prometheus Adapter가 변환한 `http_requests_per_second`
  기준 2~6대 (`hpa-custom-metric.yaml`)
- **Prometheus**: `kube-prometheus-stack`(Helm, in-cluster) — `monitoring` 네임스페이스, 6시간 보관 후
  Amazon Managed Prometheus(AMP)로 `remote_write`
- 클러스터 애드온(KEDA, kube-prometheus-stack, prometheus-adapter, aws-load-balancer-controller,
  ArgoCD)은 이 저장소가 아니라 Helm으로 직접 설치·관리한다 — 여기 있는 건 앱 워크로드 매니페스트뿐.

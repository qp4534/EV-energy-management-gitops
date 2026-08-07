# rul-diagnosis EKS 배포 사전 조건

배터리 진단(화재위험/SOH등급/잔여수명·매각가 3단계 에이전트 + 매도 제안서 PDF) 서비스.
`fastapi` 네임스페이스를 `apps/fastapi`와 같이 씁니다(테스트 시 Argo CD가 이 앱만 동기화해도
`CreateNamespace=true`라서 네임스페이스는 알아서 만들어집니다).

모델 파일(304MB+125MB)은 이 저장소에 없습니다. `EV-energy-management-fastapi`의
`.github/workflows/deploy-rul-diagnosis.yml`이 S3(`ev-mgmt-ai-models` 버킷)에서 받아
빌드 시점에 이미지 안에 구워 넣습니다. 이 매니페스트는 그 이미지를 그대로 씁니다.

`ANTHROPIC_API_KEY`는 선택 사항입니다(`/pipeline`에서 `include_report=true`로 Claude 종합
리포트를 받을 때만 필요 — `/report/pdf`, `/diagnose/erd`는 키 없이 동작). 쓰려면 Argo CD
동기화 전에 아래 Secret을 만들어 두세요.

```bash
kubectl create namespace fastapi --dry-run=client -o yaml | kubectl apply -f -
kubectl create secret generic rul-diagnosis-secret -n fastapi \
  --from-literal=ANTHROPIC_API_KEY='실제 키' \
  --dry-run=client -o yaml | kubectl apply -f -
```

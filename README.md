# EV-energy-management-gitops

ArgoCD가 감시하는 배포 매니페스트 저장소입니다.

## 구조
- `apps/<서비스명>/` — 서비스별 Kubernetes 매니페스트 (Deployment/Service/Ingress, kustomize)
- `argocd/` — ArgoCD Application 정의

## 배포 흐름
1. 각 서비스 리포의 GitHub Actions가 이미지를 빌드해 Docker Hub에 push
2. 같은 워크플로우가 `apps/<서비스명>/deployment.yaml`의 이미지 태그를 새 태그로 변경해 이 리포에 커밋
3. ArgoCD가 변경을 감지해 클러스터에 자동 반영 (RollingUpdate로 무중단 배포)
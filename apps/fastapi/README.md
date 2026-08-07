# FastAPI EKS 배포 사전 조건

이 애플리케이션은 클러스터 내부 전용 서비스입니다. 외부 Ingress는 만들지 않고,
Spring backend가 `http://ev-ai-inference-api.fastapi.svc.cluster.local`로 호출합니다.

Argo CD 동기화 전에 `fastapi` namespace에 아래 Secret이 있어야 합니다.

```bash
kubectl create namespace fastapi --dry-run=client -o yaml | kubectl apply -f -
kubectl create secret generic fastapi-secret -n fastapi \
  --from-literal=DATABASE_URL='postgresql+asyncpg://USER:PASSWORD@RDS_HOST:5432/DB_NAME' \
  --dry-run=client -o yaml | kubectl apply -f -
```

`DATABASE_URL`은 공유 RDS에 연결하는 FastAPI용 SQLAlchemy asyncpg URL입니다. 실제 값은
Git에 커밋하지 않습니다. FastAPI 이미지가 Docker Hub에 푸시된 뒤에 Argo CD Application을
동기화합니다.

# Twin API 인증 배포 준비

FastAPI Twin 쓰기 API는 내부 서비스 토큰을 요구하고, 사용자 조회 API와 WebSocket은
Spring Backend가 차량 권한 확인 후 발급한 단기 접근권을 요구한다.

배포 전에 `fastapi`, `backend` 두 namespace에 같은 두 값을 가진 Secret을 생성해야 한다.
값은 Git에 저장하지 않는다.

```powershell
$serviceToken = '<32자 이상의 무작위 서비스 토큰>'
$ticketSecret = '<32자 이상의 별도 무작위 서명 비밀>'

kubectl -n fastapi create secret generic twin-auth-secret `
  --from-literal=TWIN_SERVICE_TOKEN=$serviceToken `
  --from-literal=TWIN_TICKET_SECRET=$ticketSecret

kubectl -n backend create secret generic twin-auth-secret `
  --from-literal=TWIN_SERVICE_TOKEN=$serviceToken `
  --from-literal=TWIN_TICKET_SECRET=$ticketSecret
```

운영에서는 위 명령의 평문 입력 대신 AWS Secrets Manager와 External Secrets Operator를
사용하는 것을 권장한다. Secret 생성 전에는 새 Deployment를 적용하지 않는다.

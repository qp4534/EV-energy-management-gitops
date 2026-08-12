# 오토스케일링 구조 검토

## 결론

- `ev-ai-inference-api`는 차량별 30~120초 모델 이력을 프로세스 메모리에 보관하므로 현재는 단일 Pod로 운용한다.
- `rul-diagnosis`는 요청별 상태를 공유하지 않는 HTTP 추론 서비스이므로 CPU 사용률을 기준으로 KEDA가 1~6 Pod로 확장한다.
- 기존 SQS 두 큐는 애플리케이션 경로에 producer/consumer가 구현되기 전까지 오토스케일링 지표로 사용하지 않는다.

## 변경 내용

1. `apps/fastapi-eks/kustomization.yaml`에서 `keda-thermal-runaway.yaml` 적용을 제외했다.
2. `rul-diagnosis`의 KEDA 트리거를 SQS 큐 길이에서 평균 CPU 사용률 60%로 변경했다.
3. BMS API의 Deployment replica는 기존 1개를 유지한다.

## 향후 BMS API 수평 확장 조건

차량별 HGB 윈도우 이력을 Redis 등 공유 저장소로 이전하고, 여러 Pod에서 동일 차량의 연속 이력을 복원할 수 있다는 통합 테스트가 통과한 뒤에만 BMS API 오토스케일링을 다시 활성화한다.

SQS 방식을 사용할 경우에는 API Deployment가 아니라 실제로 큐 메시지를 수신·처리·삭제하는 전용 worker Deployment를 KEDA 대상으로 지정한다.

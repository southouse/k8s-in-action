## ConfigMap
- 설정 데이터 저장하는 리소스
- 하드코딩된 환경변수 같은 환경 별로 분리된 상태 값의 정의에 사용
- 존재하지 않는 configmap을 찹조하면 파드가 실행되지 않고, ref한 configmap을 생성하면 자동으로 실행
- `spec.containers.env.valueFrom.configMapKeyRef.optional: true`로 지정하면 configmap이 존재하지 않아도 Pod를 실행
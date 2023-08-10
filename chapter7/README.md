## ConfigMap
- 설정 데이터 저장하는 리소스
- 하드코딩된 환경변수 같은 환경 별로 분리된 상태 값의 정의에 사용
- 존재하지 않는 configmap을 찹조하면 파드가 실행되지 않고, ref한 configmap을 생성하면 자동으로 실행
- `spec.containers.env.valueFrom.configMapKeyRef.optional: true`로 지정하면 configmap이 존재하지 않아도 Pod를 실행
- configmap 볼륨을 사용해 각 항목을 파일로 노출 가능
- 볼륨에서 configmap을 참조하려면 `spec.containers.volumes.configMap.name` 을 지정
- `spec.containers.volumes.configMap.items.key` 값을 이용해 configmap의 키 값으로 개별 설정이 가능
- 기본적으로 configmap 볼륨의 모든 파일 권한은 644로 설정
- configmap을 사용하여 설정을 관리하면, pod를 다시 만들거나, 컨테이너를 다시 만들 필요 없이 configmap만 변경하여 설정을 업데이트할 수 있음
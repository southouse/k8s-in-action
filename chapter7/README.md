## ConfigMap
- 설정 데이터 저장하는 리소스
- 하드코딩된 환경변수 같은 환경 별로 분리된 상태 값의 정의에 사용
- 존재하지 않는 configmap을 찹조하면 파드가 실행되지 않고, ref한 configmap을 생성하면 자동으로 실행
- `spec.containers.env.valueFrom.configMapKeyRef.optional: true`로 지정하면 configmap이 존재하지 않아도 Pod를 실행
- configmap 볼륨을 사용해 각 항목을 파일로 노출 가능
- 볼륨에서 configmap을 참조하려면 `spec.containers.volumes.configMap.name` 을 지정
- `spec.containers.volumes.configMap.items.key` 값을 이용해 configmap의 키 값으로 개별 설정이 가능
- 비어 있지 않은 디렉토리에 마운트하면 기존 파일들이 숨겨지고 사용이 불가능해짐
  - `spec.containers.volumeMounts.subPath`로 해결
- 기본적으로 configmap 볼륨의 모든 파일 권한은 644로 설정
- configmap을 사용하여 설정을 관리하면, pod를 다시 만들거나, 컨테이너를 다시 만들 필요 없이 configmap만 변경하여 설정을 업데이트할 수 있음
- 내부적으로 `..data/` 라는 심볼릭 링크 디렉토리가 있고, 해당 디렉토리는 configmap이 변경되면 임시로 생성되는 디렉토리에 링크되어 있음
- configmap을 변경하면 모든 pod에 동기적으로 업데이트 되지 않기 때문에 pod가 실행 중인 경우에 변경하는건 권장되지 않음

## Secret
- 자격증명이나 암호화 키와 같은 보안이 유지되어야 하는 값을 저장하는 k8s 오브젝트
- Base64 인코딩을 사용
  - 일반 텍스트뿐만 아니라, 바이너리 값도 담을 수 있기 때문에
  - 파드에서 값을 읽을 때 자동으로 디코딩
- secret의 최대 크기는 1MB로 제한
- secret 파일을 저장할 땐 인메모리 파일시스템(tmpfs)를 사용
  - 민감한 데이터를 디스크에 저장하지 않기 위함
- secret을 환경변수로도 사용할 수 있음
  - 권장하지 않는 방법
- 프라이빗 이미지 레지스트리에서 이미지를 pull 받을 때 필요한 자격증명을 저장하는데 사용
  - `spec.imagePullSecrets` 사용
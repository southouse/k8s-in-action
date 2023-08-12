## Downward API
- 파드의 IP, 호스트 이름 등 실행 시점까지 알려지지 않은 데이터나 파드의 레이블과 같이 이미 설정된 데이터에서 사용
- 파드 자체의 메타테이터를 해당 파드 내에서 실행 중인 프로세스에 노출
- `spec.containers.env.valueFrom.fieldRef.fieldPath` 옵션으로 컨테이너 환경변수에 현재 Pod가 갖고 있는 메타데이터를 설정할 수 있음
- `spec.containers.env.valueFrom.resourceFieldRef.resource` 옵션으로 현재 Pod에 정의한 필드의 값을 환경변수로 설정할 수 있음
- `spec.volumes.downwardAPI` 옵션으로 파일을 만들어 볼륨을 마운트할 수 있음
- 볼륨 스펙에서 컨테이너의 리소스 필드를 참조할 때는 컨테이너 이름을 명시
  - `spec.volumes.downwardAPI.items.resourceFieldRef.containerName`
- 더 많은 메타데이터가 필요한 경우 Kubernetes API 서버에서 직접 가져와야 함

## Kubernetes API 서버
- `kubectl proxy`로 API 서버의 프록시를 생성할 수 있음
- `curl http://localhost:8001/apis/~~` 명령어를 이용해 각 API 리소스의 상세 내용을 볼 수 있음
- namespace를 지정하여 검색이 가능
  - ex. `curl http://localhost:8001/apis/batch/v1/namespaces/default/jobs/my-job`
- 파드 내에서 API 서버로의 통신
  - API 서버의 인증서는 `/var/run/secrets/kubernetes.io/serviceaccount/ca.crt` 경로에 있음
  - token의 파일 내용을 Authorization HTTP 헤더에 Bearer 토큰으로 넣어서 요청해야 함
- 앰버서더 패턴 형태로 하나의 파드에 kube-proxy 컨테이너를 생성하여 메인 컨테이너에서 API 서버를 호출할 수 있음
- 클라이언트 라이브러리로 API 서버에 통신할 수 있음
  - Java, Golang, Python 등 다양한 언어를 제공
  - Swagger UI도 제공
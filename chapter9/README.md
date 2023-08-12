## Replication Controller의 파드 배포
> 실습 중 `kubectl rolling-update의 삭제로 인해 일부 실습이 어려웠고, 대신 이후 Deployment에서 Rolling update를 진행할 수 있을 것으로 보임`
- Replication Controller를 이용해 버전이 있는 label를 연결해 업데이트
  - 기존 파드를 모두 삭제한 다음 새 파드를 시작
    - 다운타임 발생
  - 새로운 파드를 시작하고 정상적으로 동작하면 기존 파드를 삭제 (Blue/Green Deployment)
    - 잠시동안 원래 파드 세트의 2배수의 파드를 생성하기 때문에 리소스가 좀 더 필요
    - 엔드포인트(서비스)에서 연결된 파드의 버전을 새로운 버전으로 업데이트하면 한번에 트래픽이 변경
  - 기존의 파드를 하나씩 점진적으로 새로운 버전의 파드로 교체 (Rolling Update)
- 동일한 이미지 태그로 변경된 이미지를 푸시한 경우가 있을 수 있기 때문에 컨테이너의 스펙 중 `imagePullPolicy` 옵션에 `Always`를 설정하여 이미지가 노드에 있든 없든 항상 다운로드하도록 설정해야 함
- `kubectl rolling-update` 명령을 사용하지 않는 이유
  - 관리자가 만든 오브젝트(replication controller)를 쿠버네티스가 직접 수정하기 때문에 추적이 어려움
  - 서버가 아닌 클라이언트에서 업데이트 프로세스를 수행하기 때문에 네트워크 연결에 문제가 생겼을 때 롤백할 수 없음
  - 현재 오브젝트의 상태를 선언적으로 업데이트 하지 않음
    - 명령어로 업데이트를 하는 방식이기 때문에, 실제 오브젝트가 정의된 `.yaml` 파일에서 선언적으로 수정하는게 좋음
  - 이로 인해 deployment 오브젝트가 도입이 됨

## Deployment
- Deployment를 생성하면 replicaset 리소스가 하위 리소스로 생성됨
  - replicaset은 차세대 replication controller
- 
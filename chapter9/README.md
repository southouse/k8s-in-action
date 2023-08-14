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
```bash
kubectl rollout status deploy $(deployment_name) # rollout 상태 확인
kubectl rollout undo deploy $(deployment_name) # rollout 롤백(되돌리기)
kubectl rollout undo deploy $(deployment_name) --to-revision $(revision_number) # 특정 버전으로 rollout 롤백(되돌리기)
kubectl rollout history $(deployment_name) # 개정 이력 확인
```
- Deployment를 생성하면 replicaset 리소스가 하위 리소스로 생성됨
  - replicaset은 차세대 replication controller
- Deployment 상태를 확인하기 위해 `kubectl rollout` 이라는 명령어를 사용
- Pod template에 대해 configmap을 사용할 때 수정하면 업데이트 되지 않기 때문에 새로운 configmap을 생성하고 참조하도록 Pod template을 수정
- revision history를 갖고 있기 때문에 원하는 버전으로 롤백이 가능
  - 각 revision 정보를 갖고 있는 replicaset을 가지고 있어야 가능
- `editionHistoryLimit` 속성으로 revision history 수를 제한할 수 있음
- `spec.strategy.rollingUpdate`
  - `maxSurge`
    - 기본값은 25%
    - 업데이트 중 동시에 생성하는 파드 갯수
  - `maxUnavailable`
    - 기본값은 25%
    - 동시에 삭제할 수 있는 파드의 최대 갯수
- `spec.minReadySeconds`
  - readinessProbe와 적절하게 섞어서 사용하면 잘못된 이미지의 배포를 중단할 수 있음
  - pod의 status가 ready가 될때까지의 최소 대기 시간
  - 설정된 시간동안 새로운 버전의 파드는 트래픽을 받지 않음
- 기본적으로 롤아웃이 10분 이상 진행되지 않으면 실패한 것으로 간주
- `spec.progressDeadlineSeconds` 를 설정하여 롤아웃의 실패 시간을 설정할 수 있음
- 전략
  - recreate
    - 기존 버전의 파드를 모두 삭제하고 새 버전의 파드를 생성
    - 다운타임이 발생
  - rolling update
    - 이전 버전의 파드를 하나씩 제거하고 동시에 새 버전의 파드를 추가해 다운타임이 없음
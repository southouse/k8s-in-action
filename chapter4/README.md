## Replication Controller
- 파드가 항상 실행되도록 보장하기 때문에 노드가 사라지거나, 노드에서 파드가 사라지면 교체 파드를 **무조건** 생성
- RC의 셀렉터와 일치하는 기존 파드의 라벨을 변경하면, RC의 관리를 받지 않는 독립 파드가 됨
- RC에 속한 오작동 하는 파드를 범위 밖으로 빼내 디버깅하고 삭제하는 케이스가 있음
- 파드 템플릿을 변경하면, 기존의 파드엔 영향이 없고 나중에 새로 생성된 파드만 영향이 있음
- `--cascade=false` 옵션으로 RC를 삭제할 때 실행 중인 파드는 삭제하지 않고 유지할 수 있음
  - 현재 1.26버전에서 false 옵션은 deprecated 상태로, `--cascade=orphan` 옵션으로 대체됨
- 세가지 필수 요소
  - `label selector`: 파드의 범위를 결정
  - `replica count`: 실행할 파드의 원하는 수를 지정
  - `pod template`: 새로운 파드를 생성
- 장점
  - 파드가 항상 원하는 상태만큼 실행되게 함
  - 노드에 장애가 생기면 실행 중인 모든 파드에 대한 교체 복제본이 생성
  - 쉽게 수평 확장 가능

## Replication Set
- `Replication Controller`의 상위 버전
- 차이점은 `matchLabels` 셀렉터 부분에서 좀 더 다양한 표현식이 가능
  - ex. `app=foo` 파드 1개, `app=bar` 파드 1개, 총 2개가 있다고 가정할 때, RC는 둘 중에 하나만 지정 가능하고, RS는 라벨의 키 `app` 으로 지정이 가능해 파드를 2개 지정 가능
- `matchExpressions`
  - `In`: 지정된 값 중 하나와 일치해야 함
  - `NotIn`: 지정된 값과 일치하지 않아야 함
  - `Exists`: 지정된 키를 포함해야 함
  - `DoesNotExists`: 지정된 키를 포함하지 않아야 함
- 여러 표현식을 지정하는 경우 모든 표현식이 `true` 로 조건을 만족해야 함

## Job
- 기존 리소스와는 다르게 완료 상태를 가지고 있는 리소스로 컨테이너 내부의 프로세스가 실행되고 성공적으로 마무리되면 사라짐
- `spec.template.spec.restartPolicy` 를 명시적으로 설정해야 함
  - restartPolicy의 옵션
    - Always (default): Container들이 정상적으로 종료(zero exit code)되었더라도 재시작하는 정책 (Web 서버에 많이 사용)
    - OnFailure : Container가 비정상적으로 종료(non-zero exit code)하는 경우에만 재시작하는 정책 (Job에 많이 사용)
    - Never: Container의 exit code와 관계없이 재시작하지 않는 정책
- `spec.completions`는 완료되어야 할 Job의 개수, `spec.parallelism`는 한번에 처리할 잡의 개수 (병렬)
  - 만약 completions가 5개이고 parallelism은 2개일 때, 4개가 성공한 시기에 마지막으로 생성되는 파드 개수는 1개

## Cronjob
- `spec.startingDeadlineSeconds`으로 파드가 적어도 해당 설정 값 시간을 넘어서 실행된다고 실패로 간주함
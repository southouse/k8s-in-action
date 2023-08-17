# 파드의 컴퓨팅 리소스 관리
- 컨테이너가 필요로 하는 CPU와 메모리의 크기를 컨테이너 개별로 지정하며 파드 전체에 지정되지 않음
- `spec.containers.resources.requests`으로 요청 크기(최소)를, `spec.containers.resources.limits` 제한 크기(최대)를 지정
  - 스케줄러는 파드를 스케줄링할 때 리소스 요청 사항을 만족하는 노드를 고려
  - 스케줄러는 실제 사용량 보다 요청 사용량에 초점을 두고 스케줄링
- 요청 크기를 초과하여 배포되지 않았던 파드가 요청 크기가 확보되면 watch 메커니즘을 통해 자동으로 스케줄링
- 남은 리소스 자원의 경우에는 요청 크기에 따른 비율로 결정됨
  - ex. 총 cpu 2 cores를 사용할 수 있는 노드에서 파드 A가 cpu를 200m, 파드 B가 1000m을 요청했다면 1:5의 비율이므로, 남은 800m의 CPU 자원에 대해서 파드 A는 1/6 크기만큼, 파드 B는 5/6 크기만큼 할당될 수 있음
  - 아무 컨테이너도 남은 자원을 활용하지 않는다면, 비율과 관계 없이 최대로 리소스를 소모하는 컨테이너에게 자원을 할당하지만, 나머지 컨테이너가 자원을 활용하면 다시 비율의 크기대로 재분배
- CPU는 압축 가능한 리소스지만, 메모리는 그렇지 않아 리소스 제한이 필요
  - 컨테이너는 설정된 CPU 제한보다 많은 CPU 시간을 할당 받을 수 없지만, 메모리의 경우에는 프로세스를 종료시킴(OOMKilled)
- 리소스 요청 값 없이 제한을 설정하면, 기본적으로 요청 값은 제한 값과 동일한 값으로 설정
- 노드에 있는 모든 파드의 리소스 제한 합계는 노드 용량의 100%를 초과할 수 있음
  - 만약 이럴 경우에 노드 리소스가 전부 다 소진되면 리소스 확보를 위해 특정 컨테이너는 제거
- 컨테이너는 항상 노드의 리소스 자원을 기준으로 생성
  - JDK 1.8.192 버전 이전에는 `-Xms` 옵션으로 JVM의 최대 힙 크기를 지정하지 않으면, 노드의 총 메모리를 기준으로 최대 힙 크기를 설정하여 문제가 발생
    - 때문에 `-XX:+UseCGroupMemoryLimitForHeap` 옵션을 통해 cgroup으로 할당한 컨테이너 메모리를 제어해야 함
    - JVM 10 이상에서는 `-XX+UseContainerSupport` 옵션을 사용
  - 시스템의 CPU 수를 검색해 실행해야 할 작업 스레드 수를 결정하는 애플리케이션의 경우에는 `Downward API` 혹은 다음 파일을 읽어 cgroup 시스템에서 직접 설정된 CPU 제한을 검색해 스레드를 지정해야 함
    - `/sys/fs/cgroup/cpu/cpu.cfs_quota_us`
    - `/sys/fs/cgroup/cpu/cpu.cfs_period_us`

## 파드 QoS(Quality of Service) 클래스
- 종류
  - BestEffort (최하위 우선순위)
    - 리소스 요청과 제한이 없는 파드
    - CPU 시간을 전혀 못 받거나, 다른 파드를 위해 메모리가 필요할 때 가장 먼저 종료
    - 메모리 제한이 없기 때문에 메모리가 충분하다면 원하는 만큼 사용 가능
  - Burstable
    - `BestEffort`와 `Guaranteed` 를 제외한 모든 경우
  - Guaranteed (최상위 우선순위)
    - 조건
      - CPU와 메모리에 요청과 제한이 모두 설정돼야 함
      - 각 컨테이너에 설정돼야 함
      - 리소스 요청과 제한이 동일해야 함
    - 리소스 요청과 제한이 동일하기 때문에 추가 리소스를 사용할 수 없음
- 파드 컨테이너의 리소스 요청과 제한의 조합에서 파생
  - 요청만 설정되고 제한이 없는 경우 요청이 제한보다 작은 행
  - 제한만 설정된 경우 요청과 제한이 같은 행

  |CPU 요청 대 제한|메모리 요청 대 제한|컨테이너 QoS 클래스
  |---|---|---|
  |미설정|미설정|BestEffort|
  |미설정|요청 < 제한|Burstable|
  |미설정|요청 = 제한|Burstable|
  |요청 < 제한|미설정|Burstable|
  |요청 < 제한|요청 < 제한|Burstable|
  |요청 < 제한|요청 = 제한|Burstable|
  |요청 = 제한|요청 = 제한|Guaranteed|

- 모든 컨테이너가 동일한 QoS 클래스를 가지면 그것이 파드의 QoS이며, 적어도 한 컨테이너가 다른 컨테이너와 다른 클래스를 가지면, 파드의 QoS 클래스는 Burstable

  |컨테이너 A QoS 클래스|컨테이너 B QoS 클래스|파드 QoS 클래스
  |---|---|---|
  |BestEffort|BestEffort|BestEffort|
  |BestEffort|Burstable|Burstable|
  |BestEffort|Guaranteed|Burstable|
  |Burstable|Burstable|Burstable|
  |Burstable|Guaranteed|Burstable|
  |Guaranteed|Guaranteed|Guaranteed|

## 메모리가 부족할 때
- BesfEffort -> Burstable -> Guaranteed 순으로 종료
- 동일 QoS 클래스를 갖는 컨테이너
  - 실행중인 각 프로세스는 OOM(OutOfMemory) 점수를 갖고, 메모리 해제가 필요하면 가장 높은 점수의 프로세스가 종료
  - OOM 점수
    - 프로세스가 소비하는 가용 메모리의 비율
    - 컨테이너의 요청된 메모리와 파드 QoS 클래스를 기반으로 한 고정된 OOM 점수 조정
  - ex. 요청된 메모리의 90%를 사용하는 파드 B가 70%를 사용하는 파드 C보다 먼저 종료

## 네임스페이스별 파드에 대한 요청과 제한 설정
- `LimitRange` 리소스는 각 네임스페이스별 리소스의 최소/최대 제한을 지정하고, 명시적으로 지정하지 않은 컨테이너의 기본 리소스 요청을 지정
- LimitRanger 어드미션 컨트롤 플러그인에서 사용
- Pod 뿐만 아니라, 동일 네임스페이스의 다른 유형의 오브젝트에 적용
- `spec.limits.type: Pod` 인 경우 전체 파드의 최소 및 최대의 제한이고, 이는 파드 내의 컨테이너의 합계
- 최소, 최대, 기본값 외에도 제한 대 요청의 비율을 설정 가능
  - `spec.limits.maxLimitRequestRatio.cpu` 와 `spec.limits.maxLimitRequestRatio.memory` 로 설정
  - ex. CPU maxLimitRequestRatio값이 4라면, CPU 제한이 CPU 요청보다 4배 이상 큰 값이 될 수 없음

## 네임스페이스별 총 리소스 제한
- `ResourceQuota`는 네임스페이스에서 사용 가능한 총 리소스 양을 제한
- ResourceQuota는 파드를 생성할 때 적용되므로, ResourceQuota 오브젝트를 생성한 뒤에 생성된 파드에만 영향을 미침
  - 기존 파드에는 영향을 미치지 않음
- PV의 용량을 제한할 수 있음
- 오브젝트의 수를 제한하도록 구성할 수 있음
- ~~replicaset, job, deployment,~~ ingress 등과 같은 다른 오브젝트는 아직 제한될 수 없음 (2023-08-17 확인)

## Pod의 특정 상태나 QoS 클래스로 리소스 제한
- BestEffort, NotBestEffort, Terminating, NotTerminating, PriorityClass, CrossNamespacePodAffinity 총 6개의 범위로 사용
- BestEffort, NotBestEffort
  - BestEffort QoS 클래스 또는 다른 두 클래스 중 하나가 있는 파드에 적용되는지 여부를 결정
  - Terminating, NotTerminating (유효한 `activeDeadlineSeconds`가 없는)
    - 종료 과정의 파드에는 적용되지 않음

## Pod 리소스 사용량 모니터링
- Kubelet 자체에 있는 `cAdvisor` 라는 에이전트에서 리소스 사용량을 수집
- cAdvisor 메트릭을 수집하고 힙스터(현재는 metrics-server)로 전송
- `kubectl top ${RESOURCE_NAME}` 으로 메트릭을 확인
- cAdvisor와 metrics-server는 짧은 기간의 리소스 사용량 데이터를 보유해서 따로 데이터 저장을 위한 도구(InfluxDB 등)와 시각화와 분석을 위한 도구(Grafana 등)을 이용
- 
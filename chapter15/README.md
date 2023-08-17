# 파드와 클러스터 노드의 오토스케일링
## 수평적 파드 오토스케일링
- Horizontal 컨트롤러에 의해 수행되며, HorizontalPodAutoscaler(HPA)리소스를 작성해 활성화하고 원하는 대로 설정
- 프로세스
  - 확장 가능한 리소스 오브젝트를 관리하는 모든 파드의 메트릭을 수집
  - 메트릭을 지정한 목표 값과 같거나 가깝도록 하기 위해 필요한 파드 수를 계산
  - 확장 가능한 리소스의 replicas 필드를 갱신
- metrics-server에 REST를 통해 질의해 모든 파드의 메트릭을 가져옴
- 여러 파드 메트릭을 사용하는 경우
  - 각 메트릭의 레플리카 수를 개별적으로 계산한 뒤 가장 높은 값을 레플리카 수로 설정
- 메트릭 데이터가 전파돼 재조정 작업이 수행되기까지는 시간이 필요

### CPU 사용률 기반 스케일링
- 오토스케일러는 실제 CPU 사용량과 CPU 요청을 비교하기 때문에 직간접적으로 LimitRange 오브젝트를 통해 CPU 요청을 설정해야 함

### 메모리 소비량 기반 스케일링
- CPU 기반 오토스케일링에 비해 문제가 많음
  - 스케일 업 후에 오래된 파드는 어떻게든 메모리를 해제해야 하는데 이건 애플리케이션에서 관리하기 때문

### 사용자 정의 메트릭 기반 스케일링
- `spec.mertics.type: Resource`
- `spec.mertics.type: Pod`
  - QPS(초당 질의 수) 또는 메세지 브로커의 큐 메세지 수 등이 있음
- `spec.mertics.type: Object`
  - 파드에 직접 관련되지 않는 메트릭을 기반
    - ex. ingress 오브젝트의 latencyMillis 메트릭을 사용하여 스케일링하도록 설정

- 오토스케일링에 적합한 메트릭은 레플리카가 n개로 확장됐을때 어떤 메트릭 X 값이 X/n에 가깝게 줄어들어야 함
  - ex. QPS나 웹 애플리케이션의 경우에는 애플리케이션이 초당 받는 요청 수 등
- 수평 파드 오토스케일러는 최소 크기를 0으로 지정할 수 없기 때문에 어쩌다가 한번 요청을 받는 서비스의 경우에는 유휴상태로 둬야 함
  - 현재 k8s는 이 기능을 제공하지 않아, 서드파티 애플리케이션을 사용해야 함

## 수직적 파드 오토스케일링
> 책에서는 아직 베타 버전으로 정식 출시가 되지 않았다고 하지만, 현재 `VerticalPodAutoscaler` 라는 이름의 오브젝트로 사용이 가능
```bash
# Install
git clone https://github.com/kubernetes/autoscaler.git
cd autoscaler/vertical-pod-autoscaler
./hack/vpa-up.sh

# Example
cd autoscaler/vertical-pod-autoscaler/examples
kubectl apply -f hamster.yaml
```
# 클러스터 노드와 네트워크 보안
## 파드에서 호스트 리눅스 네임스페이스 사용
- 파드가 가상 네트워크 어댑터 대신 노드의 실제 네트워크 어댑터를 사용하기를 원하면 `spec.hostNetwork: true`로 설정
- hostPort를 사용하는 파드의 경우 
  - 노드포트에 대한 연결은 실행 중인 파드로 직접 전달
  - 노드 당 Pod 인스턴스 하나만 스케줄링 가능
    - 노드에 동일한 hostPort를 가진 파드가 있을 수 없기 때문
- NodePort의 서비스와 연결된 파드의 경우 임의의 파드로 전달
- `spec.hostPID: true`로 설정하면 파드의 컨테이너에서 실행 중인 프로세스가 노드의 다른 프로세스를 볼 수 있음
- `spec.hostIPC: true`로 설정하면 파드의 컨테이너에서 실행 중인 프로세스가 노드의 다른 프로세스에 IPC로 통신이 가능

## 컨테이너의 보안 컨텍스트 구성
- 컨테이너의 프로세스를 실행할 사용자 지정
  - `spec.containers.securityContext.runAsUser` 옵션에 사용자 ID를 지정
- 컨테이너가 루트로 실행되는 것을 방지
    - `spec.containers.securityContext.runAsNonRoot: true` 옵션을 지정하면 루트가 아닌 사용자로 실행
      - 악의적인 사용자가 컨테이너 이미지의 User를 root로 변경했을 때 해당 옵션을 주면 Pod의 컨테이너는 루트로 실행되며, Pod는 실행되지 않음
- 컨테이너를 Privileged mode에서 실행해 노드의 커널에 관한 모든 접근 권한을 부여
  - `spec.containers.securityContext.privileged: true` 옵션을 지정
    - 라즈베리파이 Pod가 LED를 제어하기를 원한다면 privileged 모드 사용
- 컨테이너를 Privileged mode에서 실행해 권한을 추가하거나 삭제해 세분화된 권한 구성
  - `spec.containers.securityContext.capabilities.add` 옵션으로 특정 컨테이너 권한만 추가
  - `spec.containers.securityContext.capabilities.drop` 옵션으로 특정 컨테이너 권한만 삭제
- SELinux 옵션 설정
- 프로세스가 컨테이너의 파일 시스템에 쓰기 방지
  - `spec.containers.securityContext.readOnlyRootFilesystem: true` 옵션을 설정하면 Pod 컨테이너에 마운트된 파일시스템 외에는 쓰기가 불가능

## 파드 수준의 보안 컨텍스트 구성
- 기본적으로 `spec.securityContext` 속성에서 설정
- Pod의 각 컨테이너가 다른 사용자로 실행될 때 볼륨을 공유하려면 `spec.securityContext.fsGroup` 및 `spec.securityContext.supplementalGroups`를 지정
  - 마운트한 볼륨에 대해서 fsGroup으로 설정된 그룹 ID가 지정되고, 그 외 파일시스템에 파일을 생성하면 컨테이너 내의 사용자가 속해 있던 그룹으로 생성됨

## PodSecurityPolicy
> kubenetes 1.21 버전부터 Deprecated, 1.25 버전부터 완전히 삭제되고 [PSA](https://kubernetes.io/ko/docs/tutorials/security/ns-level-pss/#%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0-%EC%A0%84%EC%97%90)로 대체
- `PodSecurityPolicy` 리소스에 구성된 정책을 유지하는 작업은 API 서버에서 실행되는 어드미션 컨트롤러 플러그인으로 수행
- 할 수 있는 작업
  - 파드가 호스트의 IPC, PID 또는 네트워크 네임스페이스를 사용할 수 있는지
  - 파드가 바인딩할 수 있는 호스트 포트
  - 컨테이너가 실행할 수 있는 사용자 ID
  - Privileged container가 있는 파드를 만들 수 있는지
  - 어떤 커널 기능이 허용되며 기본으로 추가되거나 혹은 항상 삭제되어야 하는지
  - 컨테이너가 사용할 수 있는 SELinux 레이블
  - 컨테이너가 쓰기 가능한 루트 파일시스템을 사용할 수 있는지
  - 컨테이너가 실행할 수 있는 파일시스템 그룹
  - 파드가 사용할 수 있는 볼륨 유형

## 네임스페이스에서 네트워크 격리 사용
- 일부 클라이언트 파드만 서버 파드에 연결하도록 허용하기 위해서 `NetworkPolicy` 오브젝트를 사용
- ingress와 egress로 구성할 수 있음
- 서비스 파드가 있어도 상관 없이 허용된 policy 외에는 접근할 수 없음
- 특정 네임스페이스의 라벨에 대해서도 네트워크 격리가 가능
- CIDR를 이용한 규칙도 설정 가능
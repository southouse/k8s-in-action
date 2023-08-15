## Service Account
- 각 namespace 마다 default 서비스 어카운트가 자동으로 생성
- 파드는 같은 namespace의 서비스 어카운트만 사용 가능
- 파드에서 제어할 수 있는 리소스 범위를 지정하여 클러스터를 보안 -> RBAC 플러그인을 사용
  - ex. A라는 서비스 어카운트는 리소스의 메타데이터만 검색(Get)할 수 있고, B라는 서비스 어카운트는 리소스를 생성(Create)할 수 있는 권한을 갖게 됨
- 서비스 어카운트에 파드가 마운트 가능한 Secret 오브젝트를 설정할 수 있음
  - 이 기능을 사용하려면 `kubernetes.io/enforce-mountable-secrets=true` 어노테이션을 포함해야 함
- `imagePullSecrets`에 도커 secret을 지정하여 프라이빗 이미지 레포지토리에서 컨테이너 이미지를 가져올 수 있는 설정을 할 수 있음

## RBAC 플러그인
- 인가 규칙
  - Role과 ClusterRole: 리소스에 수행할 수 있는 동사를 지정
  - RoleBinding과 ClusterRoleBinding: 위의 롤을 특정 사용자, 그룹 또는 서비스 어카운트에 바인딩
  - Cluster~~는 네임스페이스를 지정하지 않는 클러스터 수준의 리소스
  - 네임스페이스가 지정된 RoleBinding은 ClusterRole을 참조할 수 있음
- 새로 생성한 네임스페이스의 기본 서비스 어카운트로는 리소스를 가져오거나(Get) 나열(List)할 수 없음
- 하나의 rolebinding는 여러 개의 서비스 어카운트를 가질 수 있고, 하나의 role만 가질 수 있음
- 다른 네임스페이스에 생성된 role도 부여 가능
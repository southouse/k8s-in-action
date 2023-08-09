## Volume
> `minikube` 환경에서 실습을 진행하기 때문에, 책에서 일부 클라우드 제공자 전용 스토리지를 사용하는 부분과 상이할 수 있음

- 독립적인 쿠버네티스 오브젝트가 아니므로, 자체적으로 생성, 삭제될 수 없음
- 유형
  - `emptyDir`
    - 일시적인 데이터 저장
    - 메모리를 사용하는 `tmpfs` 파일시스템으로 생성하려면 `spec.volumes.emptyDir.medium: Memory`
  - `hostPath`
    - 워커 노드의 파일시스템을 파드로 마운트
    - 노드의 파일시스템이기 때문에 파드가 종료되어도 데이터가 삭제되지 않음
    - 노드의 스케줄링에 민감
    - 파드의 데이터 저장용으로 사용하는 방식은 권장되지 않음
  - Deprecated ~~`gitRepo`: Git 콘텐츠를 체크아웃해 초기화해 마운트~~
  - `nfs`
  - `awsElasticBlockStore`: 클라우드 제공자의 전용 스토리지를 마운트
  - `cinder`, `iscsi`, `vsphereVolume` 등: 기타 다른 유형의 스토리지
  - `configMap`, `secret`, `downwardAPI`: k8s 리소스나 클러스터 정보를 파드에 노출하는데 사용하는 특별한 유형의 볼륨
  - `persistentVolumeClaim`
    - 동적으로 프로비저닝된 PV를 사용하는 방법
    - PV, PVC를 사용하면 개발자 입장에서 기저에 사용된 실제 스토리지 기술을 알 필요가 없이 사용이 가능
    - `spec.persistentVolumeReclaimPolicy: Retain` 옵션이 설정되어 있으면, PVC를 삭제하고 다시 만들 때 마운트가 되지 않음
    - `spec.storageClassName`을 빼면 기본 sc로, 옵션에 "" 공백을 넣으면 sc를 사용하지 않음
- Storage Class
  - PV를 동적으로 프로비저닝할 수 있게 해줌
  - 온프레미스의 경우에는 사용자 정의 프로비저너가 필요
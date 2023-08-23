# 쿠버네티스의 확장
## 사용자 정의 API 오브젝트
- 현재 지원하는 리소스보다 훨씬 더 높은 수준의 사용자 정의 오브젝트가 생성
- 새로운 리소스를 정의하기 위해선 `CustomResourceDefinition` 오브젝트를 생성
  - CRD로 어떤 작업을 수행하도록 하려면 컨트롤러도 배포해야 함
- 컨트롤러를 직접 작성하여 사용자 정의 오브젝트에 대한 모든 것을 제어할 수 있음
  - ex. `Website` 오브젝트에 대한 컨트롤러를 작성할 때, 해당 오브젝트는 `Deployment`, `Service` 를 작성하여 웹 사이트를 배포하는 형태로 처리할 수 있음
  - proxy 오브젝트 URL에 `watch=true`를 설정하면 해당 오브젝트가 현재 어떤 API 동작을 수행하고 있는지 트래킹이 가능
    - ex. `curl http://localhost:8001/apis/extensions.example.com/v1/websites\?watch\=true`를 요청하면 아래와 같은 응답을 리턴
      ```json
        {"type":"ADDED","object":{"apiVersion":"extensions.example.com/v1","kind":"Website","metadata":{"annotations":{"kubectl.kubernetes.io/last-applied-configuration":"{\"apiVersion\":\"extensions.example.com/v1\",\"kind\":\"Website\",\"metadata\":{\"annotations\":{},\"name\":\"kubia\",\"namespace\":\"default\"},\"spec\":{\"gitRepo\":\"https://github.com/luksa/kubia-website-example.git\"}}\n"},"creationTimestamp":"2023-08-23T03:40:29Z","generation":1,"managedFields":[{"apiVersion":"extensions.example.com/v1","fieldsType":"FieldsV1","fieldsV1":{"f:metadata":{"f:annotations":{".":{},"f:kubectl.kubernetes.io/last-applied-configuration":{}}},"f:spec":{".":{},"f:gitRepo":{}}},"manager":"kubectl-client-side-apply","operation":"Update","time":"2023-08-23T03:40:29Z"}],"name":"kubia","namespace":"default","resourceVersion":"220145","uid":"5d35db79-1b22-4cee-ba5b-eaabc80ac60e"},"spec":{"gitRepo":"https://github.com/luksa/kubia-website-example.git"}}}

        {"type":"DELETED","object":{"apiVersion":"extensions.example.com/v1","kind":"Website","metadata":{"annotations":{"kubectl.kubernetes.io/last-applied-configuration":"{\"apiVersion\":\"extensions.example.com/v1\",\"kind\":\"Website\",\"metadata\":{\"annotations\":{},\"name\":\"kubia\",\"namespace\":\"default\"},\"spec\":{\"gitRepo\":\"https://github.com/luksa/kubia-website-example.git\"}}\n"},"creationTimestamp":"2023-08-23T03:40:29Z","generation":1,"managedFields":[{"apiVersion":"extensions.example.com/v1","fieldsType":"FieldsV1","fieldsV1":{"f:metadata":{"f:annotations":{".":{},"f:kubectl.kubernetes.io/last-applied-configuration":{}}},"f:spec":{".":{},"f:gitRepo":{}}},"manager":"kubectl-client-side-apply","operation":"Update","time":"2023-08-23T03:40:29Z"}],"name":"kubia","namespace":"default","resourceVersion":"220509","uid":"5d35db79-1b22-4cee-ba5b-eaabc80ac60e"},"spec":{"gitRepo":"https://github.com/luksa/kubia-website-example.git"}}}
      ```
  - 컨트롤러를 쿠버네티스 환경 안에서 실행 (보통 `Deployment` 형태로)
  - 감시 이벤트에서 오브젝트를 수신할 때 오브젝트의 유효성을 검증하는 기능을 추가해야 유효성 검증이 가능
    - `CustomResourceValidation` 오브젝트로 해결이 가능

## 사용자 정의 API 서버
- API 서버 애그리게이션(aggregation)으로 사용자 정의 API 서버를 기본 쿠버네티스 API 서버와 통합할 수 있음

## 서비스 카탈로그
- 서비스 카탈로그의 일반 API
  - `ClusterServiceBroker`: 서비스를 프로비저닝할 수 있는 시스템을 기술
  - `ClusterServiceClass`: 프로비저닝할 수 있는 서비스 유형을 기술
  - `ServiceInstance`: 프로비저닝된 서비스의 한 인스턴스
  - `ServiceBinding`: 클라이언트 파드 세트와 ServiceInstance 간의 바인딩을 기술
- 구성 요소
  - 서비스 카탈로그 API 서버
  - etcd 스토리지
  - 모든 컨트롤러를 실행하는 컨트롤러 매니저
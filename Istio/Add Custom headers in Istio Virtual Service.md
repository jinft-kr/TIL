# Add Custom headers in Istio Virtual service

### 목적
지금 당장 Frontend 에서 Notion API 를 이용하여 개발을 해야하는데, Backend의 인력 투입이 어려운 시점이었다.<br>
따라서 서비스 Frontend 에서 서비스 Backend 로직을 타지 않고 Notion API를 조회하여 원하는 응답을 받고 싶다는 요청이 들어왔다.<br>

### 해결 방법
`Notion API` 를 사용하기 위해서는 `http header` 에 `요구하는 값`들을 전달해줘야한다.<br>
우리는 `Istio` 내 서비스를 구축하여 운영하고 있었고, 인프라 단에서 `특정 path` 로 요청이 왔을 때 `header` 에 값을 넣어 요청이 가도록 설정하였다. <br>
이를 구현하기 위해서 `Istio VirtualService` 의 설정 파일을 수정하여 요구 사항을 만족 시켰다.

### [VirtualService Headers](https://istio.io/latest/docs/reference/config/networking/virtual-service/#Headers)
`Envoy` 가 `destination host` 서비스에 `요청` 또는 `응답` 을 전달할 때 `Message Header` 를 `조작` 할 수 있다. <br>
<div align="center">

|                              Field                               |            Type            |                 Description                 | 
|:--------------:|:---------------:|:-------------------------------------------:|
|                          request                         | HeaderOperations | Destination Host 에 요청을 전달하기 전에 적용할 헤더 조작 규칙 |
| response | HeaderOperations |     Caller 에게 응답을 반환하기 전에 적용할 헤더 조작 규칙      |
</div>

- [Headers.HeaderOperations](https://istio.io/latest/docs/reference/config/networking/virtual-service/#Headers-HeaderOperations)
    - `set` : 키로 지정된 헤더를 주어진 값으로 덮어쓴다.
    - `add` : 키로 지정된 헤더에 주어진 값을 추가한다(쉼표로 구분된 값 목록 생성).
    - `remove` : 지정된 헤더 제거
  
### Source
```
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: ${NAME}
  namespace: ${NAMESPACE}
spec:
  gateways:
  - ${NAMESPACE}/${GATEWAY_NAME}
  hosts:
  - ${HOST_NAME}
  http:
  - headers:
      request:
        set:
          ${KEY}: ${VALUE}
    match:
    - uri:
        prefix: /api/notion
    rewrite:
      uri: /api/notion
    route:
    - destination:
        host: server
        port:
          number: 3000
  - match:
    - uri:
        prefix: /api
    route:
    - destination:
        host: server
        port:
          number: 3000
  - match:
    - uri:
        prefix: /
    route:
    - destination:
        host: webapp
        port:
          number: 8080
```
### istio 리소스 이해하기(Gateway)

### [Gateway](https://istio.io/latest/docs/reference/config/networking/gateway/)
- HTTP/TCP 연결을 수신하는 메쉬 로드벨런서
- 노출되어야하는 포트 집합
- 사용할 프로토콜 유형
- 로드밸런서 SNI 구성, TLS 등을 설명
- envoy로 치면 listener와 Route domain쪽
- `노풀될 호스트와 포트매핑이 메인
- ```
  apiVersion: networking.istio.io/v1alpha3
  kind: Gateway
  metadata:
    name: my-gateway
    namespace: some-config-namespace
  spec:
    selector:
      app: my-gateway-controller
    servers:
    - port:
        number: 80
        name: http
        protocol: HTTP
      hosts:
      - uk.bookinfo.com
      - eu.bookinfo.com
      tls:
        httpsRedirect: true # sends 301 redirect for http requests
    - port:
        number: 443
        name: https-443
        protocol: HTTPS
      hosts:
      - uk.bookinfo.com
      - eu.bookinfo.com
      tls:
        mode: SIMPLE # enables HTTPS on this port
        serverCertificate: /etc/certs/servercert.pem
        privateKey: /etc/certs/privatekey.pem
    - port:
        number: 9443
        name: https-9443
        protocol: HTTPS
      hosts:
      - "bookinfo-namespace/*.bookinfo.com"
      tls:
        mode: SIMPLE # enables HTTPS on this port
        credentialName: bookinfo-secret # fetches certs from Kubernetes secret
    - port:
        number: 9080
        name: http-wildcard
        protocol: HTTP
      hosts:
        - "*"
    - port:
        number: 2379 # to expose internal service via external port 2379
        name: mongo
        protocol: MONGO
      hosts:
      - "*"
    ```
  
### Gateway 조심해야 할 것
- L7구성보다 L4구성에 초점이 있다고 생각하는게 편함
- 노출할 도메인들에 대해서 Gateway끼리 충돌하지 않도록 잘 설정
- 와일드 카드도메인도 좋지만 사내 namespace 분리 정책에 따라 명확히 define하는게 더 좋다.
```
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"networking.istio.io/v1alpha3","kind":"VirtualService","metadata":{"annotations":{},"name":"bookinfo","namespace":"default"},"spec":{"gateways":["bookinfo-gateway"],"hosts":["*"],"http":[{"match":[{"uri":{"exact":"/productpage"}},{"uri":{"prefix":"/static"}},{"uri":{"exact":"/login"}},{"uri":{"exact":"/logout"}},{"uri":{"prefix":"/api/v1/products"}}],"route":[{"destination":{"host":"productpage","port":{"number":9080}}}]}]}}
  creationTimestamp: "2022-07-07T09:02:24Z"
  generation: 1
  name: bookinfo
  namespace: default
  resourceVersion: "92512000"
  uid: 87237776-b3b2-4654-960d-46b2910b0b1e
spec:
  gateways:
  - bookinfo-gateway
  hosts:
  - '*'
  http:
  - match:
    - uri:
        exact: /productpage
    - uri:
        prefix: /static
    - uri:
        exact: /login
    - uri:
        exact: /logout
    - uri:
        prefix: /api/v1/products
    route:
    - destination:
        host: productpage
        port:
          number: 9080
```

```
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"networking.istio.io/v1alpha3","kind":"Gateway","metadata":{"annotations":{},"name":"bookinfo-gateway","namespace":"default"},"spec":{"selector":{"istio":"ingressgateway"},"servers":[{"hosts":["*"],"port":{"name":"http","number":80,"protocol":"HTTP"}}]}}
  creationTimestamp: "2022-07-07T09:02:24Z"
  generation: 1
  name: bookinfo-gateway
  namespace: default
  resourceVersion: "92511996"
  uid: e0396f6a-550c-4b0b-af06-1bc7ebf90a5a
spec:
  selector:
    istio: ingressgateway
  servers:
  - hosts:
    - '*'
    port:
      name: http
      number: 80
      protocol: HTTP
```

- kubectl port-forward pod/istio-ingressgateway-f7cdsdsd-21dsda 15000:15000 -n istio-system
- istio ingress gateway pod 15000 port를 localhost 15000 port와 연결을 한다.
- http://localhost:15000/config_dump 에 dynamic_route_config를 통해 어떻게 라우팅 되어있는지 알 수 있음
  - ![istio-gateway](https://user-images.githubusercontent.com/63401132/178492826-50d49089-e8fa-4270-b206-0b0d29bb9ed4.jpeg)

### Gateway 조심해야 할 것2
- VirtualService와 바인딩 상태 꼭 확인
- VirtualService에 multiple GW를 바인딩할 수 있으나 추천하진 않음. mesh + 1 개 총 두 개가 좋은 경

### Mesh Gateway
- 모든 이스티오 배포는 `숨겨진 메쉬 게이트웨이라는 암묵적인 Gateway` 를 가진다!
- 이러한 종류의 GW는 메시의 모든 서비스 프록시의 워크로드를 반영하며 모든 포트에 와일드카드 호스트를 노출
- VS가 Gateway를 `바인딩하지 않으면` 자동으로 mesh GW에 적용됨
- 즉, 메쉬 게이트웨이는 모든 사이드카

### Host 기반 트래픽 라우팅 설정
- Istio Gateway
- Istio VirtualService
- Istio DestinationRule
- 실제 트래픽 경로는?
  - ![traffic-flow](https://user-images.githubusercontent.com/63401132/178492769-9ce5ec50-50ef-417f-a26f-55cc94b85a98.jpeg)


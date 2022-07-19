### [ Issue ]
- wild card certificate를 발급받아서 같은 인증서를 통해 통신하는 서비스들 중에서
- 최초에 접속한 서비스는 통신에 성공하는데 그 다음에 같은 인증서를 사용하는 다른 서비스를 통신하면,
- 통신이 안되는 문제가 발생하였다.

### [ Problem Solution Approach ]
- 기존에는 서비스별로 네임스페이스를 만들었고, 네임스페이스 만큼 게이트웨이도 같이 생성되었었다.
- 기존 Gateway.yaml
```
kind: Gateway
apiVersion: networking.istio.io/v1beta1
metadata:
  labels:
    service: ${var.namespace}
  name: gateway
  namespace: ${var.namespace}
spec:
  selector:
    istio: ingress
  servers:
  - hosts:
    - ${var.namespace}.sample.com
    port:
      name: http
      number: 80
      protocol: HTTP
    tls:
      httpsRedirect: true
  - hosts:
    - ${var.namespace}.sample.com
    port:
      name: https
      number: 443
      protocol: HTTPS
    tls:
      credentialName: star-sample-com-tls-chain
      mode: SIMPLE
```
- 기존 VirtualService.yaml
```
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  labels:
    service: ${var.namespace}
  name: virtual-service
  namespace: ${var.namespace}
spec:
  gateways:
  - gateway
  hosts:
  - ${var.namespace}.sample.com
  http:
  - match:
    - uri:
        prefix: /api
    route:
    - destination:
        host: ${var.server}
        port:
          number: ${var.server_port}
  - match:
    - uri:
        prefix: /
    route:
    - destination:
        host: ${var.webapp}
        port:
          number: ${var.webapp_port}
```

- 따라서 하나의 게이트웨이만을 가지도록 변경하였고, destination rule을 통해 라우팅을 설정하였다.
- 변경된 Gateway.yaml
```
kind: Gateway
apiVersion: networking.istio.io/v1beta1
metadata:
  labels:
    service: common
  name: gateway
  namespace: common
spec:
  selector:
    istio: ingress
  servers:
  - hosts:
    - *.sample.com
    - sample.com
    port:
      name: http
      number: 80
      protocol: HTTP
    tls:
      httpsRedirect: true
  - hosts:
    - *.sample.com
    - sample.com
    port:
      name: https
      number: 443
      protocol: HTTPS
    tls:
      credentialName: star-sample-com-tls-chain
      mode: SIMPLE
```
- 변경된 VirtualService.yaml
```
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  labels:
    service: ${var.namespace}
  name: virtual-service
  namespace: ${var.namespace}
spec:
  gateways:
  - common/gateway
  hosts:
  - ${var.namespace}.sample.com
  http:
  - match:
    - uri:
        prefix: /api
    route:
    - destination:
        host: ${var.server}
        port:
          number: ${var.server_port}
  - match:
    - uri:
        prefix: /
    route:
    - destination:
        host: ${var.webapp}
        port:
          number: ${var.webapp_port}
```

### [Reference]
- [404 errors occur when multiple gateways configured with same TLS certificate](https://istio.io/latest/docs/ops/common-problems/network-issues/#404-errors-occur-when-multiple-gateways-configured-with-same-tls-certificate)
- [Http Connection Reuse](https://httpwg.org/specs/rfc7540.html#reuse)

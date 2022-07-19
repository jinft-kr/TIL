### 02. istio 리소스 이해하기(VirtualService)

- [VirtualService](https://istio.io/latest/docs/reference/config/networking/virtual-service/)
  - ```
    apiVersion: networking.istio.io/v1alpha3
    kind: VirtualService
    metadata:
      name: reviews-route
    spec:
      hosts:
      - reviews.prod.svc.cluster.local
      http:
      - name: "reviews-v2-routes"
        match:
        - uri:
            prefix: "/wpcatalog"
        - uri:
            prefix: "/consumercatalog"
        rewrite:
          uri: "/newcatalog"
        route:
        - destination:
            host: reviews.prod.svc.cluster.local
            subset: v2
      - name: "reviews-v1-route"
        route:
        - destination:
            host: reviews.prod.svc.cluster.local
            subset: v1    
    ```  - spec.hosts에 처리할 트래픽 hosts 등록
  - L7 패쓰별 라우팅
  - 목적지에 대한 정책이 들어가는 곳이기 때문에 매우 중요한 리소스
  - `envoy의 RouteConfig 역할을 메인으로 가져가는 리소스
    - ```
        route_config:
          name: local_route
          virtual_hosts:
          - name: local_service
            domains: ["*"]
            routes:
            - match:
                prefix: "/"
              route:
                host_rewrite_literal: www.envoyproxy.io
                cluster: service_envoyproxy_io
      ```
  - envoy를 virtualservice의 custom resource로 정의되어 있다.
- Host기반 트래픽 라우팅 설정
  - Istio Gateway
  - Istio VirtualService
  - Istio DestinationRule

kubectl exec -it productpage-v1-6b745fds-pfz98 -c istio-proxy
curl -XPOST localhost:15000/logging?http=debug
kubectl exec -it productpage-v1-6b745fds-pfz98 -c istio-proxy
health check log 들어오는거 확인

- Headers
- HttpRoute
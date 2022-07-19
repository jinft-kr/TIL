### Envoy 용어 정리

### Istio Control Plane Architecture
![Istio Control Plane](https://user-images.githubusercontent.com/63401132/177302190-dbd766eb-0e52-41f8-8f48-a16a5d836bc2.jpeg)
- control plane이 각 envoy proxy에 configuration, service discovery 등 을 동작하며 주기적인 sync를 통해 통신한다.
- control plane = istiod(istio daemon)
- data plane = istio-proxy(envoy)
- 서비스 매쉬 구현체

### Envoy Proxy
- C++로 구현된 고성능 프록시
- `네트워크 투명성`을 목표
- 다양한 필터체인 지원
- L3/L4 필터
- HTTP L7 필터
- 동적 configuration API 제공
- 기능이 많다
- CNCF / `벤더가 없다!`

### Upstream / Downstream
![Upstream-Downstream](https://user-images.githubusercontent.com/63401132/177303137-de13b201-7c7b-43d7-90c7-480034e03e05.jpeg)
- Upstream
  - envoy가 요청을 포워딩해서 연결하는 백엔드 네트워크 노드
  - sidecar일 땐 application app, 아닐 땐 원격 백엔드
- Downstream
  - An entity connecting to envoy
  - This may be local application, or a network node
  - In non-sidecar models this is a remote client

### Cluster / Endpoint
- Cluster : envoy가 트래픽을 포워드할 수 있는 논리적인 서비스(엔드포인트 세트)
  - ex) outbound|80|ingress-nginx-http-public-defaultbackend.nginx-ingress
- Endpoint: like ip address, 네트워크 노드로 클러스터로 그룹핑됨
  - ex) 10.128.74.120:8080

### Listener / Route
- Listener : 무엇을 받을지 그리고 어떻게 처리할지
  - IP/port에 바인딩하고(TCP Connection 관리), 요청 처리 측면에서 다운스트림을 조정하는 역할
- Routes: 어디로 트래픽을 보낼지

### Filter
- Filter: 리스터로부터 서비스에 트래픽을 전달하기까지 요청 처리 파이프라인
  - ![filter](https://user-images.githubusercontent.com/63401132/177303734-7ecf6cd6-1f1a-484f-967b-3ac675cef91a.jpeg)

### [Envoy proxy docs](https://www.envoyproxy.io/docs.html)
- endpoint, cluster, route, listener, filter 등과 같은 용어를 알아둘 것
```
static_resources:
  listeners:
  # There is a single listener bound to port 443.
  - name: listener_https
    address:
      socket_address:
        protocol: TCP
        address: 0.0.0.0
        port_value: 443
    # A single listener filter exists for TLS inspector.
    listener_filters:
    - name: "envoy.filters.listener.tls_inspector"
      typed_config:
        "@type": type.googleapis.com/envoy.extensions.filters.listener.tls_inspector.v3.TlsInspector
    # On the listener, there is a single filter chain that matches SNI for acme.com.
    filter_chains:
    - filter_chain_match:
        # This will match the SNI extracted by the TLS Inspector filter.
        server_names: ["acme.com"]
      # Downstream TLS configuration.
      transport_socket:
        name: envoy.transport_sockets.tls
        typed_config:
          "@type": type.googleapis.com/envoy.extensions.transport_sockets.tls.v3.DownstreamTlsContext
          common_tls_context:
            tls_certificates:
            - certificate_chain: {filename: "certs/servercert.pem"}
              private_key: {filename: "certs/serverkey.pem"}
      filters:
      # The HTTP connection manager is the only network filter.
      - name: envoy.filters.network.http_connection_manager
        typed_config:
          "@type": type.googleapis.com/envoy.extensions.filters.network.http_connection_manager.v3.HttpConnectionManager
          stat_prefix: ingress_http
          use_remote_address: true
          http2_protocol_options:
            max_concurrent_streams: 100
          # File system based access logging.
          access_log:
          - name: envoy.access_loggers.file
            typed_config:
              "@type": type.googleapis.com/envoy.extensions.access_loggers.file.v3.FileAccessLog
              path: "/var/log/envoy/access.log"
          # The route table, mapping /foo to some_service.
          route_config:
            name: local_route
            virtual_hosts:
            - name: local_service
              domains: ["acme.com"]
              routes:
              - match:
                  path: "/foo"
                route:
                  cluster: some_service
          # CustomFilter and the HTTP router filter are the HTTP filter chain.
          http_filters:
          # - name: some.customer.filter
          - name: envoy.filters.http.router
            typed_config:
              "@type": type.googleapis.com/envoy.extensions.filters.http.router.v3.Router
  clusters:
  - name: some_service
    # Upstream TLS configuration.
    transport_socket:
      name: envoy.transport_sockets.tls
      typed_config:
        "@type": type.googleapis.com/envoy.extensions.transport_sockets.tls.v3.UpstreamTlsContext
    load_assignment:
      cluster_name: some_service
      # Static endpoint assignment.
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: 10.1.2.10
                port_value: 10002
        - endpoint:
            address:
              socket_address:
                address: 10.1.2.11
                port_value: 10002
    typed_extension_protocol_options:
      envoy.extensions.upstreams.http.v3.HttpProtocolOptions:
        "@type": type.googleapis.com/envoy.extensions.upstreams.http.v3.HttpProtocolOptions
        explicit_http_config:
          http2_protocol_options:
            max_concurrent_streams: 100
  - name: some_statsd_sink
  # The rest of the configuration for statsd sink cluster.
# statsd sink.
stats_sinks:
- name: envoy.stat_sinks.statsd
  typed_config:
    "@type": type.googleapis.com/envoy.config.metrics.v3.StatsdSink
    tcp_cluster_name: some_statsd_sink
```

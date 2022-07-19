### Evovy Proxy 가볍게 맛보기

### Evovy Install
- [Get Stared](https://www.envoyproxy.io/docs/envoy/v1.22.2/start/install)
- ```
  brew update
  brew install envoy
  ```
  
### RUN Envoy
- [RUN Envoy](https://www.envoyproxy.io/docs/envoy/v1.22.2/start/quick-start/run-envoy)
- `envoy -c envoy-demo.yaml`
  - envoy-demo.yaml
      ```
      static_resources:
    
        listeners:
        - name: listener_0
          address:
            socket_address:
              address: 0.0.0.0
              port_value: 10000
          filter_chains:
          - filters:
            - name: envoy.filters.network.http_connection_manager
              typed_config:
                "@type": type.googleapis.com/envoy.extensions.filters.network.http_connection_manager.v3.HttpConnectionManager
                stat_prefix: ingress_http
                access_log:
                - name: envoy.access_loggers.stdout
                  typed_config:
                    "@type": type.googleapis.com/envoy.extensions.access_loggers.stream.v3.StdoutAccessLog
                http_filters:
                - name: envoy.filters.http.router
                  typed_config:
                    "@type": type.googleapis.com/envoy.extensions.filters.http.router.v3.Router
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
    
        clusters:
        - name: service_envoyproxy_io
          type: LOGICAL_DNS
          # Comment out the following line to test on v6 networks
          dns_lookup_family: V4_ONLY
          load_assignment:
            cluster_name: service_envoyproxy_io
            endpoints:
            - lb_endpoints:
              - endpoint:
                  address:
                    socket_address:
                      address: www.envoyproxy.io
                      port_value: 443
          transport_socket:
            name: envoy.transport_sockets.tls
            typed_config:
              "@type": type.googleapis.com/envoy.extensions.transport_sockets.tls.v3.UpstreamTlsContext
              sni: www.envoyproxy.io
      ```
- `localhost:10000`
  - ![localhost_envoy](https://user-images.githubusercontent.com/63401132/177318754-188465a3-189a-4768-80ef-db87510b3fba.jpeg)
- `envoy -c envoy-demo.yaml --config-yaml "$(cat envoy-override.yaml)"`
  - envoy-override.yaml
    - ```
      admin:
        address:
          socket_address:
            address: 127.0.0.1
            port_value: 9902
      ```
- `localhost:9902`
  - ![envoy_admin](https://user-images.githubusercontent.com/63401132/177319541-5a728520-59cb-4321-82ae-da29a9c9c3d3.jpeg)
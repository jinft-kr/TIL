### Envoy xDS 이해하기(Dynamic Api Configuration)

### istioctl proxy-status
- ![istioctl_proxy-status](https://user-images.githubusercontent.com/63401132/177321244-4073dd51-29c6-425f-b5db-4b682a0252b3.jpeg)
  - CDS : Cluster Discovery Service
  - LDS : Listener Discovery Service
  - EDS : Endpoint Discovery Service
  - RDS : Router Discovery Service

### x Discovery Service
- 동적으로 각 영역의 설정을 로드하는 방법
- Management 서비스와 통신하면서 configuration을 갱신
- gRPC, REST 방식 모두 지원


### Envoy와 Management Server와 통신하는 Network Flow
- Envoy와 Management Server와 통신하는 Network Flow
  - ![Envoy_Management_server_network_flow](https://user-images.githubusercontent.com/63401132/177322027-7acaaabe-46be-4eaf-925a-be2a2f20d8c8.jpeg)
- ADS를 통해 다양한 타입의 리소스를 싱글 스트림에서 해결
  - ADS: Aggregation Deiscobery Service
- [Common discovery API components](https://www.envoyproxy.io/docs/envoy/v1.22.2/api-v3/service/discovery/v3/discovery.proto.html?highlight=common%20discovery)
- [xDS REST and gRPC protocol](https://www.envoyproxy.io/docs/envoy/latest/api-docs/xds_protocol)

### istio proxy configuration
- istio-proxy container에 접근하여 /etc/istio/proxt/envoy-rev0.json 에서 envoy 설정을 확인할 수 있음
  - ![envoy_configuration](https://user-images.githubusercontent.com/63401132/177323759-143aeea1-5ca7-4f64-88fc-5847f3e700a0.jpeg)
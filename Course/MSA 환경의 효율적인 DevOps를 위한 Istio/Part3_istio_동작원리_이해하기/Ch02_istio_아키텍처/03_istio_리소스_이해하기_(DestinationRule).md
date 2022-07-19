### istio 리소스 이해하기(DestinationRUle)

### [DestinationRule](https://istio.io/latest/docs/reference/config/networking/destination-rule/)
- 클라이언트 사이드 Load Balancing
- 서비스 엔드포인트를 비정상으로 표시하는 조건
- L4, L7 커넥션 풀 설정
- 서버의 TLS 설정
- envoy로 치면 cluster, endpoint 쪽
- ```
  apiVersion: networking.istio.io/v1alpha3
  kind: DestinationRule
  metadata:
    name: bookinfo-ratings
  spec:
    host: ratings.prod.svc.cluster.local
    trafficPolicy:
      loadBalancer:
        simple: LEAST_REQUEST
    subsets:
    - name: testversion
      labels:
        version: v3
      trafficPolicy:
        loadBalancer:
          simple: ROUND_ROBIN
    ```
- 
    
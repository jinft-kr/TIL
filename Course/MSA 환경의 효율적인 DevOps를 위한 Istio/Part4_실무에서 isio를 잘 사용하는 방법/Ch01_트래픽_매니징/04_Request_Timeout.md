### 03. Request Timeout
- 요청한 시간 내에 트래픽이 오지 않으면 요청이 처리되도록 하는 것

### [Request Timeout](https://istio.io/latest/docs/tasks/traffic-management/request-timeouts/)
- `kubectl apply -f samples/bookinfo/networking/virtual-service-all-v1.yaml`
  - ```
    (istio-study-cluster:default)➜  istio-1.14.1 kubectl apply -f samples/bookinfo/networking/virtual-service-all-v1.yaml
    W0720 21:25:09.739166   91998 gcp.go:120] WARNING: the gcp auth plugin is deprecated in v1.22+, unavailable in v1.25+; use gcloud instead.
    To learn more, consult https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke
    virtualservice.networking.istio.io/productpage unchanged
    virtualservice.networking.istio.io/reviews configured
    virtualservice.networking.istio.io/ratings unchanged
    virtualservice.networking.istio.io/details unchanged
    ```
- v2: 검정 별 rating 만들기
  - ```
    $ kubectl apply -f - <<EOF
    apiVersion: networking.istio.io/v1alpha3
    kind: VirtualService
    metadata:
      name: reviews
    spec:
      hosts:
        - reviews
      http:
      - route:
        - destination:
            host: reviews
            subset: v2
    EOF
    ```
  - 모든 요청이 v2로 가는걸 확인할 수 있음
- 2초 후에 요청이 가도록 셋팅
  - ```
     $ kubectl apply -f - <<EOF
     apiVersion: networking.istio.io/v1alpha3
     kind: VirtualService
     metadata:
       name: ratings
     spec:
       hosts:
       - ratings
       http:
       - fault:
           delay:
             percent: 100
             fixedDelay: 2s
         route:
         - destination:
             host: ratings
             subset: v1
     EOF
     ```
  - request가 2초 후에 오는 것을 확인 할 수 있음
- 0.5초 timeout 셋팅을 주면 다시 tcp 커넥션이 될 것임
  - ```
    kubectl apply -f - <<EOF
    apiVersion: networking.istio.io/v1alpha3
    kind: VirtualService
    metadata:
      name: reviews
    spec:
      hosts:
      - reviews
      http:
      - route:
        - destination:
            host: reviews
            subset: v2
        timeout: 0.5s
    EOF
    ```
  - 1초 후에 request가 온 것을 알 수 있음
- 실제로 timeout이 발생해서 다시 connection을 통해 request가 들어올까?
  - [istio productpage github](https://github.com/istio/istio/blob/master/samples/bookinfo/src/productpage/productpage.py)
    - ```
      def getProductReviews(product_id, headers):
        # Do not remove. Bug introduced explicitly for illustration in fault injection task
        # TODO: Figure out how to achieve the same effect using Envoy retries/timeouts
        for _ in range(2):
            try:
                url = reviews['name'] + "/" + reviews['endpoint'] + "/" + str(product_id)
                res = requests.get(url, headers=headers, timeout=3.0)
            except BaseException:
                res = None
            if res and res.status_code == 200:
                return 200, res.json()
        status = res.status_code if res is not None and res.status_code else 500
        return status, {'error': 'Sorry, product reviews are currently unavailable for this book.'}
      ```
      - 반복문 2번 호출해서 retry되도록 하드코딩이 되어 있는걸 확인할 수 있음
  - productpage istioproxy container accesslog를 확인해보면 504 error가 2번 발생하고 그 다음 200 응답이 오는 것을 확인할 수 있다.
    - `kubectl logs productpage-v1-7795568889-j4dhl -c istio-proxy -f` 
      - ```
        [2022-07-20T12:35:56.198Z] "GET /reviews/0 HTTP/1.1" 504 UT upstream_response_timeout - "-" 0 24 500 - "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/103.0.0.0 Safari/537.36" "7d15bcc7-284f-965c-9306-9ae81c27c8cb" "reviews:9080" "10.0.2.12:9080" outbound|9080|v2|reviews.default.svc.cluster.local 10.0.2.10:45648 10.4.11.161:9080 10.0.2.10:37026 - -
        [2022-07-20T12:35:56.705Z] "GET /reviews/0 HTTP/1.1" 504 UT upstream_response_timeout - "-" 0 24 500 - "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/103.0.0.0 Safari/537.36" "7d15bcc7-284f-965c-9306-9ae81c27c8cb" "reviews:9080" "10.0.2.12:9080" outbound|9080|v2|reviews.default.svc.cluster.local 10.0.2.10:45662 10.4.11.161:9080 10.0.2.10:37040 - -
        [2022-07-20T12:35:56.178Z] "GET /productpage HTTP/1.1" 200 - via_upstream - "-" 0 3992 1031 1030 "10.0.2.1" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/103.0.0.0 Safari/537.36" "7d15bcc7-284f-965c-9306-9ae81c27c8cb" "34.64.193.56" "10.0.2.10:9080" inbound|9080|| 127.0.0.6:49409 10.0.2.10:9080 10.0.2.1:0 outbound_.9080_._.productpage.default.svc.cluster.local default
        2022-07-20T12:35:57.350234Z     debug   envoy http      [C266417] new stream
        2022-07-20T12:35:57.350384Z     debug   envoy http      [C266417][S12200732740557894846] request headers complete (end_stream=true):
        ':authority', '34.64.193.56'
        ':path', '/static/bootstrap/css/bootstrap-theme.min.css.map'
        ':method', 'GET'
        'pragma', 'no-cache'
        'cache-control', 'no-cache'
        'user-agent', 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/103.0.0.0 Safari/537.36'
        'accept-encoding', 'gzip, deflate'
        'accept-language', 'ko-KR,ko;q=0.9,en-US;q=0.8,en;q=0.7'
        'cookie', 'session=eyJ1c2VyIjoiamFzb24ifQ.YtbDgw.-9g_96ySPiFCkMxHL4l2Sck8Wso'
        'x-forwarded-for', '10.0.2.1'
        'x-forwarded-proto', 'http'
        'x-envoy-internal', 'true'
        'x-request-id', '09236b7e-60d5-9c21-b722-cf708e877e0e'
        'x-envoy-decorator-operation', 'productpage.default.svc.cluster.local:9080/static*'
        'x-envoy-peer-metadata', 'ChQKDkFQUF9DT05UQUlORVJTEgIaAAoaCgpDTFVTVEVSX0lEEgwaCkt1YmVybmV0ZXMKGgoMSU5TVEFOQ0VfSVBTEgoaCDEwLjAuMi41ChkKDUlTVElPX1ZFUlNJT04SCBoGMS4xNC4xCr8DCgZMQUJFTFMStAMqsQMKHQoDYXBwEhYaFGlzdGlvLWluZ3Jlc3NnYXRld2F5ChMKBWNoYXJ0EgoaCGdhdGV3YXlzChQKCGhlcml0YWdlEggaBlRpbGxlcgo2CilpbnN0YWxsLm9wZXJhdG9yLmlzdGlvLmlvL293bmluZy1yZXNvdXJjZRIJGgd1bmtub3duChkKBWlzdGlvEhAaDmluZ3Jlc3NnYXRld2F5ChkKDGlzdGlvLmlvL3JldhIJGgdkZWZhdWx0CjAKG29wZXJhdG9yLmlzdGlvLmlvL2NvbXBvbmVudBIRGg9JbmdyZXNzR2F0ZXdheXMKIQoRcG9kLXRlbXBsYXRlLWhhc2gSDBoKNjY2OGY5NTQ4ZAoSCgdyZWxlYXNlEgcaBWlzdGlvCjkKH3NlcnZpY2UuaXN0aW8uaW8vY2Fub25pY2FsLW5hbWUSFhoUaXN0aW8taW5ncmVzc2dhdGV3YXkKLwojc2VydmljZS5pc3Rpby5pby9jYW5vbmljYWwtcmV2aXNpb24SCBoGbGF0ZXN0CiIKF3NpZGVjYXIuaXN0aW8uaW8vaW5qZWN0EgcaBWZhbHNlChoKB01FU0hfSUQSDxoNY2x1c3Rlci5sb2NhbAovCgROQU1FEicaJWlzdGlvLWluZ3Jlc3NnYXRld2F5LTY2NjhmOTU0OGQtNXMya3AKGwoJTkFNRVNQQUNFEg4aDGlzdGlvLXN5c3RlbQpdCgVPV05FUhJUGlJrdWJlcm5ldGVzOi8vYXBpcy9hcHBzL3YxL25hbWVzcGFjZXMvaXN0aW8tc3lzdGVtL2RlcGxveW1lbnRzL2lzdGlvLWluZ3Jlc3NnYXRld2F5Cr4DChFQTEFURk9STV9NRVRBREFUQRKoAyqlAwpIChBnY3BfZ2NlX2luc3RhbmNlEjQaMmdrZS1pc3Rpby1zdHVkeS1jbHVzdGVyLWRlZmF1bHQtcG9vbC04NTg3YWJlZS1zZ3B0CiwKE2djcF9nY2VfaW5zdGFuY2VfaWQSFRoTNTk2ODcyMzI0ODQ4MTk4NTYzMAotChRnY3BfZ2tlX2NsdXN0ZXJfbmFtZRIVGhNpc3Rpby1zdHVkeS1jbHVzdGVyCo4BChNnY3BfZ2tlX2NsdXN0ZXJfdXJsEncadWh0dHBzOi8vY29udGFpbmVyLmdvb2dsZWFwaXMuY29tL3YxL3Byb2plY3RzL3Blbmd1aW4tZmx5LWRldi9sb2NhdGlvbnMvYXNpYS1ub3J0aGVhc3QzLWEvY2x1c3RlcnMvaXN0aW8tc3R1ZHktY2x1c3RlcgojCgxnY3BfbG9jYXRpb24SExoRYXNpYS1ub3J0aGVhc3QzLWEKIAoLZ2NwX3Byb2plY3QSERoPcGVuZ3Vpbi1mbHktZGV2CiQKEmdjcF9wcm9qZWN0X251bWJlchIOGgw0NDEwNDAzMDM1MzQKJwoNV09SS0xPQURfTkFNRRIWGhRpc3Rpby1pbmdyZXNzZ2F0ZXdheQ=='
        'x-envoy-peer-metadata-id', 'router~10.0.2.5~istio-ingressgateway-6668f9548d-5s2kp.istio-system~istio-system.svc.cluster.local'
        'x-envoy-attempt-count', '1'
        'x-b3-traceid', '361df7d2953ffa2a5d61ad9cb1f205ae'
        'x-b3-spanid', '5d61ad9cb1f205ae'
        'x-b3-sampled', '1'
    
        2022-07-20T12:35:57.350404Z     debug   envoy http      [C266417][S12200732740557894846] request end stream
        2022-07-20T12:35:57.350996Z     debug   envoy http      [C266418] new stream
        2022-07-20T12:35:57.351098Z     debug   envoy http      [C266418][S14421955729045122438] request headers complete (end_stream=true):
        ':authority', '34.64.193.56'
        ':path', '/static/bootstrap/css/bootstrap.min.css.map'
        ':method', 'GET'
        'pragma', 'no-cache'
        'cache-control', 'no-cache'
        'user-agent', 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/103.0.0.0 Safari/537.36'
        'accept-encoding', 'gzip, deflate'
        'accept-language', 'ko-KR,ko;q=0.9,en-US;q=0.8,en;q=0.7'
        'cookie', 'session=eyJ1c2VyIjoiamFzb24ifQ.YtbDgw.-9g_96ySPiFCkMxHL4l2Sck8Wso'
        'x-forwarded-for', '10.0.2.1'
        'x-forwarded-proto', 'http'
        'x-envoy-internal', 'true'
        'x-request-id', '65447416-0e18-977b-97e5-a1bcad384837'
        'x-envoy-decorator-operation', 'productpage.default.svc.cluster.local:9080/static*'
        'x-envoy-peer-metadata', 'ChQKDkFQUF9DT05UQUlORVJTEgIaAAoaCgpDTFVTVEVSX0lEEgwaCkt1YmVybmV0ZXMKGgoMSU5TVEFOQ0VfSVBTEgoaCDEwLjAuMi41ChkKDUlTVElPX1ZFUlNJT04SCBoGMS4xNC4xCr8DCgZMQUJFTFMStAMqsQMKHQoDYXBwEhYaFGlzdGlvLWluZ3Jlc3NnYXRld2F5ChMKBWNoYXJ0EgoaCGdhdGV3YXlzChQKCGhlcml0YWdlEggaBlRpbGxlcgo2CilpbnN0YWxsLm9wZXJhdG9yLmlzdGlvLmlvL293bmluZy1yZXNvdXJjZRIJGgd1bmtub3duChkKBWlzdGlvEhAaDmluZ3Jlc3NnYXRld2F5ChkKDGlzdGlvLmlvL3JldhIJGgdkZWZhdWx0CjAKG29wZXJhdG9yLmlzdGlvLmlvL2NvbXBvbmVudBIRGg9JbmdyZXNzR2F0ZXdheXMKIQoRcG9kLXRlbXBsYXRlLWhhc2gSDBoKNjY2OGY5NTQ4ZAoSCgdyZWxlYXNlEgcaBWlzdGlvCjkKH3NlcnZpY2UuaXN0aW8uaW8vY2Fub25pY2FsLW5hbWUSFhoUaXN0aW8taW5ncmVzc2dhdGV3YXkKLwojc2VydmljZS5pc3Rpby5pby9jYW5vbmljYWwtcmV2aXNpb24SCBoGbGF0ZXN0CiIKF3NpZGVjYXIuaXN0aW8uaW8vaW5qZWN0EgcaBWZhbHNlChoKB01FU0hfSUQSDxoNY2x1c3Rlci5sb2NhbAovCgROQU1FEicaJWlzdGlvLWluZ3Jlc3NnYXRld2F5LTY2NjhmOTU0OGQtNXMya3AKGwoJTkFNRVNQQUNFEg4aDGlzdGlvLXN5c3RlbQpdCgVPV05FUhJUGlJrdWJlcm5ldGVzOi8vYXBpcy9hcHBzL3YxL25hbWVzcGFjZXMvaXN0aW8tc3lzdGVtL2RlcGxveW1lbnRzL2lzdGlvLWluZ3Jlc3NnYXRld2F5Cr4DChFQTEFURk9STV9NRVRBREFUQRKoAyqlAwpIChBnY3BfZ2NlX2luc3RhbmNlEjQaMmdrZS1pc3Rpby1zdHVkeS1jbHVzdGVyLWRlZmF1bHQtcG9vbC04NTg3YWJlZS1zZ3B0CiwKE2djcF9nY2VfaW5zdGFuY2VfaWQSFRoTNTk2ODcyMzI0ODQ4MTk4NTYzMAotChRnY3BfZ2tlX2NsdXN0ZXJfbmFtZRIVGhNpc3Rpby1zdHVkeS1jbHVzdGVyCo4BChNnY3BfZ2tlX2NsdXN0ZXJfdXJsEncadWh0dHBzOi8vY29udGFpbmVyLmdvb2dsZWFwaXMuY29tL3YxL3Byb2plY3RzL3Blbmd1aW4tZmx5LWRldi9sb2NhdGlvbnMvYXNpYS1ub3J0aGVhc3QzLWEvY2x1c3RlcnMvaXN0aW8tc3R1ZHktY2x1c3RlcgojCgxnY3BfbG9jYXRpb24SExoRYXNpYS1ub3J0aGVhc3QzLWEKIAoLZ2NwX3Byb2plY3QSERoPcGVuZ3Vpbi1mbHktZGV2CiQKEmdjcF9wcm9qZWN0X251bWJlchIOGgw0NDEwNDAzMDM1MzQKJwoNV09SS0xPQURfTkFNRRIWGhRpc3Rpby1pbmdyZXNzZ2F0ZXdheQ=='
        'x-envoy-peer-metadata-id', 'router~10.0.2.5~istio-ingressgateway-6668f9548d-5s2kp.istio-system~istio-system.svc.cluster.local'
        'x-envoy-attempt-count', '1'
        'x-b3-traceid', 'f4b433a4263e1650d7c733eff9a80660'
        'x-b3-spanid', 'd7c733eff9a80660'
        'x-b3-sampled', '1'
    
        2022-07-20T12:35:57.351113Z     debug   envoy http      [C266418][S14421955729045122438] request end stream
        2022-07-20T12:35:57.357339Z     debug   envoy http      [C266418][S14421955729045122438] encoding headers via codec (end_stream=false):
        ':status', '404'
        'content-type', 'text/html'
        'content-length', '232'
        'server', 'istio-envoy'
        'date', 'Wed, 20 Jul 2022 12:35
        ```
  - reviews v2 accesslog를 확인해보자
    - `kubectl logs reviews-v2-858f99c99-6wcg7 -c istio-proxy -f`
      - ```
        [2022-07-20T12:34:35.400Z] "GET /reviews/0 HTTP/1.1" 0 DC downstream_remote_disconnect - "-" 0 0 497 - "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/103.0.0.0 Safari/537.36" "a9f0f4d6-fd4d-9ecd-a4ec-fd03ff8b0774" "reviews:9080" "10.0.2.12:9080" inbound|9080|| 127.0.0.6:38479 10.0.2.12:9080 10.0.2.10:44922 outbound_.9080_.v2_.reviews.default.svc.cluster.local default
        [2022-07-20T12:34:35.908Z] "GET /reviews/0 HTTP/1.1" 0 DC downstream_remote_disconnect - "-" 0 0 496 - "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/103.0.0.0 Safari/537.36" "a9f0f4d6-fd4d-9ecd-a4ec-fd03ff8b0774" "reviews:9080" "10.0.2.12:9080" inbound|9080|| 127.0.0.6:60857 10.0.2.12:9080 10.0.2.10:44932 outbound_.9080_.v2_.reviews.default.svc.cluster.local default
        [2022-07-20T12:34:35.418Z] "GET /ratings/0 HTTP/1.1" 200 DI via_upstream - "-" 0 48 2003 3 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/103.0.0.0 Safari/537.36" "a9f0f4d6-fd4d-9ecd-a4ec-fd03ff8b0774" "ratings:9080" "10.0.2.7:9080" outbound|9080|v1|ratings.default.svc.cluster.local 10.0.2.12:51240 10.4.7.100:9080 10.0.2.12:37794 - -
        ```
        - 2번  downstream_remote_disconnect 이 발생되었고 그 다음 connection이 성공된 걸 확인할 수 있다.

### [envoy accesslog format](https://www.envoyproxy.io/docs/envoy/latest/configuration/observability/access_log/usage)
- TCP Connection을 확인하고 싶을 때 envoy accesslog format을 custom해서 원하는 로그도 찍어볼 수 있음
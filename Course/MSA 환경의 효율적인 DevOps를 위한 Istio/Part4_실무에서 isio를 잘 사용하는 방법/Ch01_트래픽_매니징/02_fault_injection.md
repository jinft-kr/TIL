### 02. fault injection
- 특정 API에 대해서 fault로 막아야되는 경우 제어하는데 사용

### [Fault injection](https://istio.io/latest/docs/tasks/traffic-management/fault-injection/)
- `kubectl apply -f samples/bookinfo/networking/virtual-service-ratings-test-delay.yaml `
  - ```
    (istio-study-cluster:default)➜  istio-1.14.1 kubectl apply -f samples/bookinfo/networking/virtual-service-ratings-test-delay.yaml 
    W0719 23:35:56.048616   66440 gcp.go:120] WARNING: the gcp auth plugin is deprecated in v1.22+, unavailable in v1.25+; use gcloud instead.
    To learn more, consult https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke
    virtualservice.networking.istio.io/ratings configured
    ```
  - ```
    apiVersion: networking.istio.io/v1alpha3
    kind: VirtualService
    metadata:
      name: ratings
    spec:
      hosts:
      - ratings
      http:
      - match:
        - headers:
            end-user:
              exact: jason
        fault:
          delay:
            percentage:
              value: 100.0
            fixedDelay: 7s
        route:
        - destination:
            host: ratings
            subset: v1
      - route:
        - destination:
            host: ratings
            subset: v1
    ```
  - jason이라는 User가 호출을 할 때 7초의 딜레이를 준다
  - ![delay-ratings](https://user-images.githubusercontent.com/63401132/179779853-324d0dd7-b322-41ef-8a0c-0162d90b8e1d.jpeg)

### [Injecting an HTTP abort fault](https://istio.io/latest/docs/tasks/traffic-management/fault-injection/#injecting-an-http-abort-fault)
- `kubectl apply -f samples/bookinfo/networking/virtual-service-ratings-test-abort.yaml`
  - ```
    (istio-study-cluster:default)➜  istio-1.14.1 kubectl apply -f samples/bookinfo/networking/virtual-service-ratings-test-abort.yaml
    
    W0719 23:48:35.578614   66970 gcp.go:120] WARNING: the gcp auth plugin is deprecated in v1.22+, unavailable in v1.25+; use gcloud instead.
    To learn more, consult https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke
    virtualservice.networking.istio.io/ratings configured
    ```
  - ```
    apiVersion: networking.istio.io/v1alpha3
    kind: VirtualService
    metadata:
      name: ratings
    spec:
      hosts:
      - ratings
      http:
      - match:
        - headers:
            end-user:
              exact: jason
        fault:
          abort:
            percentage:
              value: 100.0
            httpStatus: 500
        route:
        - destination:
            host: ratings
            subset: v1
      - route:
        - destination:
            host: ratings
            subset: v1
    ```
  - ![Injecting an HTTP abort fault](https://user-images.githubusercontent.com/63401132/179780866-aae55714-ec0d-4a1b-90c2-c4bda66e9374.jpeg)
- `kubectl logs reviews-v2-858f99c99-6wcg7 -c istio-proxy -f`
  - ```
    [2022-07-19T14:50:22.956Z] "GET /ratings/0 HTTP/1.1" 500 FI fault_filter_abort - "-" 0 18 0 - "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/103.0.0.0 Safari/537.36" "fde60c8a-b448-9202-b726-ec7da46a227f" "ratings:9080" "-" outbound|9080|v1|ratings.default.svc.cluster.local - 10.4.7.100:9080 10.0.2.12:46698 - -
    ```
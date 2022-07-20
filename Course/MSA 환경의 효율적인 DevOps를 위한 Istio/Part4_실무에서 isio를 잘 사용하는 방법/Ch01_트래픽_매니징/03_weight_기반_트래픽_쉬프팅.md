### 03. weight 기반 트래픽 쉬프팅
- 카나리 배포도 트래픽 쉬프팅으로 구현할 수 있다.
- v1, v3 50% -> v3 100%
### [Traffic Shifting](https://istio.io/latest/docs/tasks/traffic-management/traffic-shifting/)
- `kubectl apply -f samples/bookinfo/networking/virtual-service-all-v1.yaml`
  - ```
    (istio-study-cluster:default)➜  istio-1.14.1 kubectl apply -f samples/bookinfo/networking/virtual-service-all-v1.yaml
    W0720 21:16:09.463398   91481 gcp.go:120] WARNING: the gcp auth plugin is deprecated in v1.22+, unavailable in v1.25+; use gcloud instead.
    To learn more, consult https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke
    virtualservice.networking.istio.io/productpage unchanged
    virtualservice.networking.istio.io/reviews configured
    virtualservice.networking.istio.io/ratings configured
    virtualservice.networking.istio.io/details unchanged
    (gke_penguin-fly-dev_asia-northeast3-a_istio-study-clus
    ```
- `kubectl apply -f samples/bookinfo/networking/virtual-service-reviews-50-v3.yaml`
  - ```
    (istio-study-cluster:default)➜  istio-1.14.1 kubectl apply -f samples/bookinfo/networking/virtual-service-reviews-50-v3.yaml
    W0720 21:16:58.198368   91511 gcp.go:120] WARNING: the gcp auth plugin is deprecated in v1.22+, unavailable in v1.25+; use gcloud instead.
    To learn more, consult https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke
    virtualservice.networking.istio.io/reviews configured
    ```
  - ```
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
            subset: v1
          weight: 50
        - destination:
            host: reviews
            subset: v3
          weight: 50
    ```
  - weight가 추가된 걸 확인할 수 있음
- 비율에 따라 카나리 배포를 진행할 수 있음

### [TCP Traffic Shifting](https://istio.io/latest/docs/tasks/traffic-management/tcp-traffic-shifting/)
- tcp-echo 를 통해 traffic shifting이 잘되는지 확인할 수 있다.

### [Canary Deployments using Istio](https://istio.io/latest/blog/2017/0.1-canary/)
### 01. request 라우팅 및 응용하기
- reviews
  - v1: rating 호출 안함
  - v2: rating 검정색 별 호출
  - v3: rating 빨간 별 호출
  
### [Apply default destination rules](https://istio.io/latest/docs/examples/bookinfo/#apply-default-destination-rules)
- `kubectl apply -f samples/bookinfo/networking/destination-rule-all.yaml`
  - ```
    (istio-study-cluster:default)➜  istio-1.14.1 kubectl apply -f samples/bookinfo/networking/destination-rule-all.yaml
    W0719 22:41:51.797771   59348 gcp.go:120] WARNING: the gcp auth plugin is deprecated in v1.22+, unavailable in v1.25+; use gcloud instead.
    To learn more, consult https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke
    destinationrule.networking.istio.io/productpage created
    destinationrule.networking.istio.io/reviews created
    destinationrule.networking.istio.io/ratings created
    destinationrule.networking.istio.io/details created
    ```
  - ```
    apiVersion: networking.istio.io/v1alpha3
    kind: DestinationRule
    metadata:
      name: productpage
    spec:
      host: productpage
      subsets:
      - name: v1
        labels:
          version: v1
    ---
    apiVersion: networking.istio.io/v1alpha3
    kind: DestinationRule
    metadata:
      name: reviews
    spec:
      host: reviews
      subsets:
      - name: v1
        labels:
          version: v1
      - name: v2
        labels:
          version: v2
      - name: v3
        labels:
          version: v3
    ---
    apiVersion: networking.istio.io/v1alpha3
    kind: DestinationRule
    metadata:
      name: ratings
    spec:
      host: ratings
      subsets:
      - name: v1
        labels:
          version: v1
      - name: v2
        labels:
          version: v2
      - name: v2-mysql
        labels:
          version: v2-mysql
      - name: v2-mysql-vm
        labels:
          version: v2-mysql-vm
    ---
    apiVersion: networking.istio.io/v1alpha3
    kind: DestinationRule
    metadata:
      name: details
    spec:
      host: details
      subsets:
      - name: v1
        labels:
          version: v1
      - name: v2
        labels:
          version: v2
    ---
    ```
### [Apply a virtual service](https://istio.io/latest/docs/tasks/traffic-management/request-routing/)
- `kubectl apply -f samples/bookinfo/networking/virtual-service-all-v1.yaml`
  - ```
    (istio-study-cluster:default)➜  istio-1.14.1 kubectl apply -f samples/bookinfo/networking/virtual-service-all-v1.yaml                        
    W0719 22:45:37.605120   59571 gcp.go:120] WARNING: the gcp auth plugin is deprecated in v1.22+, unavailable in v1.25+; use gcloud instead.
    To learn more, consult https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke
    virtualservice.networking.istio.io/productpage created
    virtualservice.networking.istio.io/reviews created
    virtualservice.networking.istio.io/ratings created
    virtualservice.networking.istio.io/details created
    ```
  - ```
    apiVersion: networking.istio.io/v1alpha3
    kind: VirtualService
    metadata:
      name: productpage
    spec:
      hosts:
      - productpage
      http:
      - route:
        - destination:
            host: productpage
            subset: v1
    ---
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
    ---
    apiVersion: networking.istio.io/v1alpha3
    kind: VirtualService
    metadata:
      name: ratings
    spec:
      hosts:
      - ratings
      http:
      - route:
        - destination:
            host: ratings
            subset: v1
    ---
    apiVersion: networking.istio.io/v1alpha3
    kind: VirtualService
    metadata:
      name: details
    spec:
      hosts:
      - details
      http:
      - route:
        - destination:
            host: details
            subset: v1
    ---
    ```
  - 이제는 모든 virtualservice가 v1을 바라보기때문에 rating를 호출 안할 것임
    - ![virtualservice-v1](https://user-images.githubusercontent.com/63401132/179767476-097111f3-d998-46c5-b9c3-2cd9378c4f32.jpeg)

### [Route based on user identity](https://istio.io/latest/docs/tasks/traffic-management/request-routing/)
- `kubectl apply -f samples/bookinfo/networking/virtual-service-reviews-test-v2.yaml`
  - ```
    (istio-study-cluster:default)➜  istio-1.14.1 kubectl apply -f samples/bookinfo/networking/virtual-service-reviews-test-v2.yaml
    W0719 22:56:05.856155   60178 gcp.go:120] WARNING: the gcp auth plugin is deprecated in v1.22+, unavailable in v1.25+; use gcloud instead.
    To learn more, consult https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke
    virtualservice.networking.istio.io/reviews configured
    ```
  - ```
    apiVersion: networking.istio.io/v1beta1
    kind: VirtualService
    ...
    spec:
      hosts:
      - reviews
      http:
      - match:
        - headers:
            end-user:
              exact: jason
        route:
        - destination:
            host: reviews
            subset: v2
      - route:
        - destination:
            host: reviews
            subset: v1
    ```
  - v2가 추가된 것을 확인할 수 있음
    - header에 end-user = jason이면 reviews v2로 라우팅
  - ![login-jason](https://user-images.githubusercontent.com/63401132/179768947-69fe7e59-c2f7-4276-bcee-c721b8546368.jpeg)
  - ![virtualservice-v2](https://user-images.githubusercontent.com/63401132/179768968-af211321-c350-4d49-a8a2-42e5c1effdcf.jpeg)
    - rating 검정별이 호출되는 것을 볼 수 있음
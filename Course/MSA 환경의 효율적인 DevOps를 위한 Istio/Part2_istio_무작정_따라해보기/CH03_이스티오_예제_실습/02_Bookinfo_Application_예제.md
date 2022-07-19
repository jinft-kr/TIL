### 02. [Bookinfo Application 예제](https://istio.io/latest/docs/examples/bookinfo/)
- Bookinfo
  - polyglot 마이크로서비스
  - productpage가 필요한 리뷰서비스와, 디테일 서비스를 호출
  - reviews는 3가지 버전이 있음
    - rating을 호출하지 않는 녀석
    - ratings를 호출하고 각 등급 - 검정별
    - ratings를 호출하고 각 등급 - 빨간별
  - ![bookinfo_application_architecture](https://user-images.githubusercontent.com/63401132/177031851-4dfd49db-782a-450c-af39-dca16a362b76.jpeg)

- Deploy the sample application
  - `kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml`
    - ```
      (istio-study-cluster:default)➜  istio-1.14.1 kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
      W0714 18:13:28.913946   94124 gcp.go:120] WARNING: the gcp auth plugin is deprecated in v1.22+, unavailable in v1.25+; use gcloud instead.
      To learn more, consult https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke
      service/details created
      serviceaccount/bookinfo-details created
      deployment.apps/details-v1 created
      service/ratings created
      serviceaccount/bookinfo-ratings created
      deployment.apps/ratings-v1 created
      service/reviews created
      serviceaccount/bookinfo-reviews created
      deployment.apps/reviews-v1 created
      deployment.apps/reviews-v2 created
      deployment.apps/reviews-v3 created
      service/productpage created
      serviceaccount/bookinfo-productpage created
      deployment.apps/productpage-v1 created
      ``` 
  - `kubectl get services`
    - ```
      (istio-study-cluster:default)➜  istio-1.14.1 kubectl get services
      W0714 18:13:37.120117   94149 gcp.go:120] WARNING: the gcp auth plugin is deprecated in v1.22+, unavailable in v1.25+; use gcloud instead.
      To learn more, consult https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke
      NAME          TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE
      details       ClusterIP   10.4.0.6      <none>        9080/TCP   8s
      kubernetes    ClusterIP   10.4.0.1      <none>        443/TCP    13m
      productpage   ClusterIP   10.4.4.241    <none>        9080/TCP   7s
      ratings       ClusterIP   10.4.7.100    <none>        9080/TCP   8s
      reviews       ClusterIP   10.4.11.161   <none>        9080/TCP   7s
      ```
  - `kubectl get pods`
    - ```
      (istio-study-cluster:default)➜  istio-1.14.1 kubectl get pods
      W0714 18:17:46.784140   94246 gcp.go:120] WARNING: the gcp auth plugin is deprecated in v1.22+, unavailable in v1.25+; use gcloud instead.
      To learn more, consult https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke
      NAME                              READY   STATUS    RESTARTS   AGE
      details-v1-7d88846999-wkw2w       2/2     Running   0          4m18s
      productpage-v1-7795568889-j4dhl   2/2     Running   0          4m16s
      ratings-v1-754f9c4975-d2jzq       2/2     Running   0          4m17s
      reviews-v1-55b668fc65-ffnxv       2/2     Running   0          4m17s
      reviews-v2-858f99c99-fwtw2        2/2     Running   0          4m17s
      reviews-v3-7886dd86b9-jw8x4       2/2     Running   0          4m17s
      ```
  - `kubectl exec "$(kubectl get pod -l app=ratings -o jsonpath='{.items[0].metadata.name}')" -c ratings -- curl -sS productpage:9080/productpage | grep -o "<title>.*</title>"`
    - ```
      (istio-study-cluster:default)➜  istio-1.14.1 kubectl exec "$(kubectl get pod -l app=ratings -o jsonpath='{.items[0].metadata.name}')" -c ratings -- curl -sS productpage:9080/productpage | grep -o "<title>.*</title>"
  
      W0714 18:19:14.750663   94361 gcp.go:120] WARNING: the gcp auth plugin is deprecated in v1.22+, unavailable in v1.25+; use gcloud instead.
      To learn more, consult https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke
      W0714 18:19:14.880223   94359 gcp.go:120] WARNING: the gcp auth plugin is deprecated in v1.22+, unavailable in v1.25+; use gcloud instead.
      To learn more, consult https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke
      <title>Simple Bookstore App</title>
      ```
- Open the application to outside traffic
  - `kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml`
    - ```
      (istio-study-cluster:default)➜  istio-1.14.1 kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml
      W0714 18:21:26.835088   94411 gcp.go:120] WARNING: the gcp auth plugin is deprecated in v1.22+, unavailable in v1.25+; use gcloud instead.
      To learn more, consult https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke
      gateway.networking.istio.io/bookinfo-gateway created
      virtualservice.networking.istio.io/bookinfo created
      ```
  - `istioctl analyze`
    - ```
      (istio-study-cluster:default)➜  istio-1.14.1 istioctl analyze
  
      ✔ No validation issues found when analyzing namespace: default.
      ```
- Determining the ingress IP and ports
  - ```
    export INGRESS_HOST=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
    export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].port}')
    export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].port}')
    ```
    - ```
      (istio-study-cluster:default)➜  istio-1.14.1 echo "$INGRESS_HOST"
      34.64.193.56
      (istio-study-cluster:default)➜  istio-1.14.1 echo "$INGRESS_PORT"
      80
      (istio-study-cluster:default)➜  istio-1.14.1 echo "$SECURE_INGRESS_PORT"
      443
      ```
  - `export GATEWAY_URL=$INGRESS_HOST:$INGRESS_PORT`
  - `echo "$GATEWAY_URL"`
    - ```
      (istio-study-cluster:default)➜  istio-1.14.1 echo "$GATEWAY_URL"
      34.64.193.56:80
      ```
- Verify external access
  - `echo "http://$GATEWAY_URL/productpage"`
    - ```
      (istio-study-cluster:default)➜  istio-1.14.1 echo "http://$GATEWAY_URL/productpage"
      http://34.64.193.56:80/productpage
      ```
  - product page가 잘 보이는지 확인
    - ![productpage](https://user-images.githubusercontent.com/63401132/178950883-345c0dd8-fabe-4fff-aa8f-04fedc84db3c.jpeg)

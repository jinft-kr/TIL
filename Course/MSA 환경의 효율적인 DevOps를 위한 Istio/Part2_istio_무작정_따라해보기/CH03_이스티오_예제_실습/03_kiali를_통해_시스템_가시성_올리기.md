### 03. kiali를 통해 시스템 가시성 올리기
- View the dashboard
    - `kubectl apply -f samples/addons`
        - ```
      (istio-study-cluster:default)➜  istio-1.14.1 kubectl apply -f samples/addons
      W0714 18:30:28.191125   94786 gcp.go:120] WARNING: the gcp auth plugin is deprecated in v1.22+, unavailable in v1.25+; use gcloud instead.
      To learn more, consult https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke
      serviceaccount/grafana created
      configmap/grafana created
      service/grafana created
      deployment.apps/grafana created
      configmap/istio-grafana-dashboards created
      configmap/istio-services-grafana-dashboards created
      deployment.apps/jaeger created
      service/tracing created
      service/zipkin created
      service/jaeger-collector created
      serviceaccount/kiali created
      configmap/kiali created
      clusterrole.rbac.authorization.k8s.io/kiali-viewer created
      clusterrole.rbac.authorization.k8s.io/kiali created
      clusterrolebinding.rbac.authorization.k8s.io/kiali created
      role.rbac.authorization.k8s.io/kiali-controlplane created
      rolebinding.rbac.authorization.k8s.io/kiali-controlplane created
      service/kiali created
      deployment.apps/kiali created
      serviceaccount/prometheus created
      configmap/prometheus created
      clusterrole.rbac.authorization.k8s.io/prometheus created
      clusterrolebinding.rbac.authorization.k8s.io/prometheus created
      service/prometheus created
      deployment.apps/prometheus created
      ```
    - `istioctl dashboard kiali`
        - ![kiali](https://user-images.githubusercontent.com/63401132/178951735-8cfc8e71-3201-4593-860c-41ea3d5ae3ea.jpeg)
    - `vi kiali.sh`
        - ```
      #! /bin/bash
      for i in $(seq 1 100); do curl -s -o /dev/null "http://$GATEWAY_URL/productpage"; done
      ```
    - `chmod +x ./kiali.sh`
    - `./kiali.sh`
        - ![kiali-traffic](https://user-images.githubusercontent.com/63401132/178954070-e8f1379d-4f1c-4482-9037-1df46275ac33.jpeg)
        - ![kiali-inbound-metric](https://user-images.githubusercontent.com/63401132/178954949-b25632c3-9acf-48ba-821e-64c33cfcc410.jpeg)
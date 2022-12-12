# Helm Install and Practice

### Helm CLI Install
- Prepare
  - Kubectl 을 실행할 수 있는 환경(= Kubernetes Cluster)
  - [Minikube](https://minikube.sigs.k8s.io/docs/start/)
    - `brew install minikube`
    - `minikube start`
- [helm cli install from script](https://helm.sh/docs/intro/install/)
  - ```
    $ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
    $ chmod 700 get_helm.sh
    $ ./get_helm.sh
    ```
  - `chmod +x get_helm.sh`
  - `helm version`
  
### Helm Chart Install
- helm install [NAME] [CHART] [flags]
  - [wordpress install practice](https://bitnami.com/stack/wordpress/helm)
    - `helm repo add bitnami https://charts.bitnami.com/bitnami`
    - `helm list`
    - `helm search repo bitnami` : helm repo bitnami에 어떤 레포가 있는지 찾을 수 있음
    - `helm install my-release bitnami/wordpress`
      - ```
        W1207 17:55:39.109299    5773 gcp.go:120] WARNING: the gcp auth plugin is deprecated in v1.22+, unavailable in v1.25+; use gcloud instead.
        To learn more, consult https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke
        I1207 17:55:40.431150    5773 request.go:601] Waited for 1.029791917s due to client-side throttling, not priority and fairness, request: GET:https://34.64.174.171/apis/artifactregistry.cnrm.cloud.google.com/v1beta1?timeout=32s
        NAME: my-release
        LAST DEPLOYED: Wed Dec  7 17:55:44 2022
        NAMESPACE: default
        STATUS: deployed
        REVISION: 1
        TEST SUITE: None
        NOTES:
        CHART NAME: wordpress
        CHART VERSION: 15.2.18
        APP VERSION: 6.1.1

        ** Please be patient while the chart is being deployed **

        Your WordPress site can be accessed through the following DNS name from within your cluster:

            my-release-wordpress.default.svc.cluster.local (port 80)

        To access your WordPress site from outside the cluster follow the steps below:

        1. Get the WordPress URL by running these commands:

        NOTE: It may take a few minutes for the LoadBalancer IP to be available.
        Watch the status with: 'kubectl get svc --namespace default -w my-release-wordpress'

        export SERVICE_IP=$(kubectl get svc --namespace default my-release-wordpress --include "{{ range (index .status.loadBalancer.ingress 0) }}{{ . }}{{ end }}")
        echo "WordPress URL: http://$SERVICE_IP/"
        echo "WordPress Admin URL: http://$SERVICE_IP/admin"

        2. Open a browser and access WordPress using the obtained URL.

           1. Login with the following credentials below to see your blog:

        echo Username: user
        echo Password: $(kubectl get secret --namespace default my-release-wordpress -o jsonpath="{.data.wordpress-password}" | base64 -d)
        ```
    - `helm ls `
      - ```
        NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                   APP VERSION
        my-release      default         1               2022-12-07 17:55:44.236171 +0900 JST    deployed        wordpress-15.2.18       6.1.1
        ```
    - `kubectl get svc` : wordpress external IP 찾기
      - ```
        NAME                   TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)                      AGE
        my-release-mariadb     ClusterIP      10.10.29.181   <none>        3306/TCP                     3m16s
        my-release-wordpress   LoadBalancer   10.10.29.75    34.64.76.56   80:32638/TCP,443:31321/TCP   3m16s
        ```
    - `external IP`로 `wordpress` 접근 해보기
      - <img width="1726" alt="image (14)" src="https://user-images.githubusercontent.com/63401132/206135067-a081c05e-dc34-4da8-841d-ec4ce09e9616.png">

### Helm Chart Upgrade
- helm upgrade [RELEASE] [CHART] [flags]
  - `helm ls`
    - ```
      NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                   APP VERSION
      my-release      default         1               2022-12-07 17:55:44.236171 +0900 JST    deployed        wordpress-15.2.18       6.1.1  
      ```
  - `kubectl get po`
    - ```
      NAME                                    READY   STATUS    RESTARTS   AGE
      my-release-mariadb-0                    2/2     Running   0          8m49s
      my-release-wordpress-58d68fb99d-vf4nk   2/2     Running   0          8m49s
      ```
  - `helm upgrade -h`
  - `vi values.yaml`
    - ```
      replicaCount : 3
      ```
  - `helm upgrade my-release bitnami/wordpress -f values.yaml`
  - `kubectl get po` : pod의 개수가 1 -> 3 으로 변경된 것을 알 수 있음
    - ```
      NAME                                    READY   STATUS     RESTARTS   AGE
      my-release-mariadb-0                    2/2     Running    0          11m
      my-release-wordpress-58d68fb99d-gg9qv   2/2     Running    0          41s
      my-release-wordpress-58d68fb99d-r2zbw   2/2     Running    0          36s
      my-release-wordpress-58d68fb99d-vf4nk   2/2     Running    0          11m
      ```
  - `helm ls` : revision 이 1 -> 2 로 변경된 것을 알 수 있음
    - ```
      NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                   APP VERSION
      my-release      default         2               2022-12-07 18:06:38.652685 +0900 JST    deployed        wordpress-15.2.18       6.1.1 
      ```
  - `kubectl get secret` : release에 따라 secret이 생성된 것을 알 수 있음(helm은 secret을 통해 revision을 관리하고, 이 secret을 통해 이전 버전으로 rollback 가능)
    - ```
      NAME                               TYPE                                  DATA   AGE
      default-token-9fqb6                kubernetes.io/service-account-token   3      41d
      my-release-mariadb                 Opaque                                2      13m
      my-release-mariadb-token-lmtss     kubernetes.io/service-account-token   3      13m
      my-release-wordpress               Opaque                                1      13m
      sh.helm.release.v1.my-release.v1   helm.sh/release.v1                    1      13m
      sh.helm.release.v1.my-release.v2   helm.sh/release.v1                    1      2m31s
      ```
  - `helm get -h` : 어떤 release가 가지고 있는 manifest, value 등을 알 수 있음
  - `helm get values my-release`
    - ```
      USER-SUPPLIED VALUES:
      replicaCount: 3
      ```
  - `helm get values my-release --revision 1` : revision 별 value를 알 수 있음
    - ```
      USER-SUPPLIED VALUES:
      null
      ```
  - `helm get values my-release --revision 2`
    - ```
      USER-SUPPLIED VALUES:
      replicaCount: 3
      ```
  - `helm plugin install https://github.com/databus23/helm-diff`
    - [helm diff](https://github.com/databus23/helm-diff) : plugin이기 때문에 별도 설치 필요
  - `helm diff revision my-release 1 2` : revision에 따른 차이를 알 수 있음

### Helm Chart Rollback
- helm rollback <RELEASE> [REVISION] [flags]
  - `helm history` 
  - `helm history my-release`
    - ```
      REVISION        UPDATED                         STATUS          CHART                   APP VERSION     DESCRIPTION     
      1               Wed Dec  7 17:55:44 2022        superseded      wordpress-15.2.18       6.1.1           Install complete
      2               Wed Dec  7 18:06:38 2022        deployed        wordpress-15.2.18       6.1.1           Upgrade complete
      ```
  - `helm rollback my-release 1 --dry-run` 
    - `--dry-run` : 실제로 수행되진 않지만 어떤 결과가 나올지 미리 보여줌
    - ```
      Rollback was a success! Happy Helming!
      ```
  - `helm rollback my-release 1`
  - `helm history my-release`
    - ```
      REVISION        UPDATED                         STATUS          CHART                   APP VERSION     DESCRIPTION     
      1               Wed Dec  7 17:55:44 2022        superseded      wordpress-15.2.18       6.1.1           Install complete
      2               Wed Dec  7 18:06:38 2022        superseded      wordpress-15.2.18       6.1.1           Upgrade complete
      3               Wed Dec  7 18:40:17 2022        deployed        wordpress-15.2.18       6.1.1           Rollback to 1 
      ```
  - `helm get values my-release --revision 3`
    - ```
      USER-SUPPLIED VALUES:
      null
      ```
  - `kubectl get po`
    - ```
      NAME                                    READY   STATUS    RESTARTS   AGE
      my-release-mariadb-0                    2/2     Running   0          46m
      my-release-wordpress-58d68fb99d-vf4nk   2/2     Running   0          46m
      ```
  - `helm rollback my-release 4`
  - `helm history my-release`
    - ```
      REVISION        UPDATED                         STATUS          CHART                   APP VERSION     DESCRIPTION     
      1               Wed Dec  7 17:55:44 2022        superseded      wordpress-15.2.18       6.1.1           Install complete
      2               Wed Dec  7 18:06:38 2022        superseded      wordpress-15.2.18       6.1.1           Upgrade complete
      3               Wed Dec  7 18:40:10 2022        superseded      wordpress-15.2.18       6.1.1           Rollback to 1   
      4               Wed Dec  7 18:43:02 2022        deployed        wordpress-15.2.18       6.1.1           Rollback to 2 
      ```
  - `kubectl get po
    - ```
      NAME                                    READY   STATUS     RESTARTS   AGE
      my-release-mariadb-0                    2/2     Running    0          11m
      my-release-wordpress-58d68fb99d-7frsv   2/2     Running    0          41s
      my-release-wordpress-58d68fb99d-sgxvs   2/2     Running    0          36s
      my-release-wordpress-58d68fb99d-vf4nk   2/2     Running    0          11m
      ```
### Helm Chart Delete
- helm delete RELEASE_NAME [...] [flags]
  - `helm delete my-release` : my-release 완전 삭제
- helm uninstall RELEASE_NAME [...] [flags]
  - `--keep-history` : 리소스는 지워지지만, `history` 는 유지할 수 있음, rollback시 유용하게 사용할 수 있음
    - `helm delete` 와의 차이점 중 하나
    - `helm delete my-release --keep-history`
    - `kubectl get secret`
      - ```
        NAME                               TYPE                                  DATA   AGE
        default-token-9fqb6                kubernetes.io/service-account-token   3      41d
        sh.helm.release.v1.my-release.v1   helm.sh/release.v1                    1      56m
        sh.helm.release.v1.my-release.v2   helm.sh/release.v1                    1      45m
        sh.helm.release.v1.my-release.v3   helm.sh/release.v1                    1      11m
        sh.helm.release.v1.my-release.v4   helm.sh/release.v1                    1      9m2s
        sh.helm.release.v1.my-release.v5   helm.sh/release.v1                    1      8m52s
        ```
    - `helm history my-release`
      - ```
        sh.helm.release.v1.my-release.v5   helm.sh/release.v1                    1      8m52s
        ```
        
### Helm Command

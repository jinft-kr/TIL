### 04. CLI 도구를 활용한 디버깅과 실무 팁

### 실무에서 사용하는 서비스 이슈 발생!!
- 트래픽이 많다면 파드별 accesslog는 꺼져있을 수 있음
- 해당 파드에 접근해 istio-proxy의 로그 on
- isdiod와 istio-proxy 로그 분석 및 프록시 상태/config 확인

### kubectl exec -it {POD} -c {container} -- {command}
- 정말 자주 사용하는 명령어
- 파드, 컨테이너에 직접 붙어볼 수 있고 docker exec 와 비슷한 맥락인데 해당 머신에 접근해야하는 비용 감소
- 특히 sh 명령어를 통해 해당 컨테이너에 접근해 디버깅 용도로 굉장히 유용함
- ```
  (istio-study-cluster:default)➜  TIL git:(main) ✗ kubectl exec -it productpage-v1-7795568889-j4dhl -- sh
  W0714 19:00:09.067194   97798 gcp.go:120] WARNING: the gcp auth plugin is deprecated in v1.22+, unavailable in v1.25+; use gcloud instead.
  To learn more, consult https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke
  $
  ```

### AccessLog
- k8s configmap istio 
  - data.mesh.accessLogFile: /dev/stdout 이 default 설정인데, accessLogFile: '' 공백으로 바꾸면 accesslog를 끌 수 있다.  
  - 엄청나게 많은 트래픽이 들어오는 경우에 accesslog를 모든 서비스에 켜놓으면 리소스 낭비를 할 수 있는데, 이를 방지 할 수 있음
  - ```
    apiVersion: v1
    data:
      mesh: |-
        accessLogFile: /dev/stdout
        defaultConfig:
          discoveryAddress: istiod.istio-system.svc:15012
          proxyMetadata: {}
          tracing:
            zipkin:
              address: zipkin.istio-system:9411
        enablePrometheusMerge: true
        extensionProviders:
        - envoyOtelAls:
            port: 4317
            service: otel-collector.istio-system.svc.cluster.local
          name: otel
        rootNamespace: istio-system
        trustDomain: cluster.local
      meshNetworks: 'networks: {}'
    kind: ConfigMap
    metadata:
      annotations:
        kubectl.kubernetes.io/last-applied-configuration: |
          {"apiVersion":"v1","data":{"mesh":"accessLogFile: /dev/stdout\ndefaultConfig:\n  discoveryAddress: istiod.istio-system.svc:15012\n  proxyMetadata: {}\n  tracing:\n    zipkin:\n      address: zipkin.istio-system:9411\nenablePrometheusMerge: true\nextensionProviders:\n- envoyOtelAls:\n    port: 4317\n    service: otel-collector.istio-system.svc.cluster.local\n  name: otel\nrootNamespace: istio-system\ntrustDomain: cluster.local","meshNetworks":"networks: {}"},"kind":"ConfigMap","metadata":{"annotations":{},"labels":{"install.operator.istio.io/owning-resource":"unknown","install.operator.istio.io/owning-resource-namespace":"istio-system","istio.io/rev":"default","operator.istio.io/component":"Pilot","operator.istio.io/managed":"Reconcile","operator.istio.io/version":"1.14.1","release":"istio"},"name":"istio","namespace":"istio-system"}}
      creationTimestamp: "2022-07-14T09:11:43Z"
      labels:
        install.operator.istio.io/owning-resource: unknown
        install.operator.istio.io/owning-resource-namespace: istio-system
        istio.io/rev: default
        operator.istio.io/component: Pilot
        operator.istio.io/managed: Reconcile
        operator.istio.io/version: 1.14.1
        release: istio
      name: istio
      namespace: istio-system
      resourceVersion: "5727"
      uid: 173a0d60-b9bb-42cc-a6f6-3312b10b23ad
    ```
    - accesslog를 끄도록 configmap을 수정하면 어떻게 될까?
      - istiod log를 확인해보면 mesh configuration이 update된 것을 확인할 수 있고, accesslog설정이 아예 보이지 않게된다.(없어진다)
        - ```
          2022-07-14T10:11:03.256031Z     info    Loaded MeshNetworks config from Kubernetes API server.
          2022-07-14T10:11:03.282952Z     info    Loaded MeshConfig config from Kubernetes API server.
          2022-07-14T10:11:03.284510Z     info    mesh configuration updated to: {
          "proxyListenPort": 15001,
          "connectTimeout": "10s",
          "protocolDetectionTimeout": "0s",
          "ingressClass": "istio",
          "ingressService": "istio-ingressgateway",
          "ingressControllerMode": "STRICT",
          "enableTracing": true,
          "defaultConfig": {
          "configPath": "./etc/istio/proxy",
          "binaryPath": "/usr/local/bin/envoy",
          "serviceCluster": "istio-proxy",
          "drainDuration": "45s",
          "parentShutdownDuration": "60s",
          "discoveryAddress": "istiod.istio-system.svc:15012",
          "proxyAdminPort": 15000,
          "controlPlaneAuthPolicy": "MUTUAL_TLS",
          ```
      - pod의 istio-proxy log를 보면 트래픽이 들어오는 로그가 찍히지 않는다는 것을 확인할 수 있다.
        - 그렇다면 특정 pod의 istio proxy accesslog는 키도록 어떻게 설정할 수 있을까?
          - `curl -XPOST localhost:15000/logging`
            - 특정 istio-proxy의 로그 디버깅모드 동적 설정
            - 현실에서는 accesslog를 키는게 대용량 트래픽때 상당한 부담이 될 수 있으므로 이슈 발생시 분석용으로 on/off
            - Admin 페이지 port forward로
            - ```
              (istio-study-cluster:default)➜  TIL git:(main) ✗ kubectl exec -it productpage-v1-7795568889-j4dhl -c istio-proxy -- sh                
              W0714 19:20:58.617869   99113 gcp.go:120] WARNING: the gcp auth plugin is deprecated in v1.22+, unavailable in v1.25+; use gcloud instead.
              To learn more, consult https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke
              $ curl -XPOST localhost:15000/logging
              active loggers:
              admin: warning
              alternate_protocols_cache: warning
              aws: warning
              assert: warning
              backtrace: warning
              cache_filter: warning
              client: warning
              config: warning
              connection: warning
              conn_handler: warning
              decompression: warning
              dns: warning
              dubbo: warning
              envoy_bug: warning
              ext_authz: warning
              ext_proc: warning
              rocketmq: warning
              file: warning
              filter: warning
              forward_proxy: warning
              grpc: warning
              happy_eyeballs: warning
              hc: warning
              health_checker: warning
              http: warning
              http2: warning
              hystrix: warning
              ...
              ```
              - 기본적으로 log가 warning으로 되어 있음
              - http low option을 debug로 변경하려면?
                - `$ curl -XPOST localhost:15000/logging?http=debug`
                  - ```
                    http: debug
                    http2: warning
                    hystrix: warning
                    init: warning
                    io: warning
                    jwt: warning
                    ```
                - 변경 후 http 로그가 들어오는 것을 확인할 수 있음
                - ```
                  [2022-07-14T10:01:36.164Z] "GET /static/bootstrap/fonts/glyphicons-halflings-regular.woff2 HTTP/1.1" 200 - via_upstream - "-" 0 18028 17 15 "10.178.0.35" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/103.0.0.0 Safari/537.36" "ee2e2f56-fa95-9fd8-b342-1d39cabf5019" "34.64.193.56" "10.0.2.10:9080" inbound|9080|| 127.0.0.6:41619 10.0.2.10:9080 10.178.0.35:0 outbound_.9080_._.productpage.default.svc.cluster.local default
                  [2022-07-14T10:11:28.109Z] "GET /productpage HTTP/1.1" 200 - via_upstream - "-" 0 5290 49 48 "10.178.0.34" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/103.0.0.0 Safari/537.36" "ecc5babb-fdba-9c4c-906d-6b448fd344b4" "34.64.193.56" "10.0.2.10:9080" inbound|9080|| 127.0.0.6:60987 10.0.2.10:9080 10.178.0.34:0 outbound_.9080_._.productpage.default.svc.cluster.local default
                  2022-07-14T10:13:02.340978Z     info    xdsproxy        connected to upstream XDS server: istiod.istio-system.svc:15012
                  2022-07-14T10:24:59.844257Z     debug   envoy http      [C3088][S4382910619673639635] encoding headers via codec (end_stream=false):
                  ':status', '200'
                  'content-type', 'text/plain; charset=UTF-8'
                  'cache-control', 'no-cache, max-age=0'
                  'x-content-type-options', 'nosniff'
                  'date', 'Thu, 14 Jul 2022 10:24:59 GMT'
                  'server', 'envoy'
                  
                  2022-07-14T10:25:01.227208Z     debug   envoy http      [C3089] new stream
                  2022-07-14T10:25:01.227364Z     debug   envoy http      [C3089][S9491976187023219926] request headers complete (end_stream=true):
                  ':authority', '10.0.2.10:15021'
                  ':path', '/healthz/ready'
                  ':method', 'GET'
                  'user-agent', 'kube-probe/1.22'
                  'accept', '*/*'
                  'connection', 'close'
                  ```

### istioctl proxy-status
- istio-proxy 들의 SYNC 전체 상태를 확인할 수 있는 명령어
  - `SYNCED`
  - `NOT SENT` 
  - `STALE`
- ```
  (istio-study-cluster:default)➜  istio-1.14.1 istioctl proxy-status
  NAME                                                   CLUSTER        CDS        LDS        EDS        RDS          ECDS         ISTIOD                      VERSION
  details-v1-7d88846999-wkw2w.default                    Kubernetes     SYNCED     SYNCED     SYNCED     SYNCED       NOT SENT     istiod-8495d444bb-w2j4p     1.14.1
  istio-egressgateway-575d8bd99b-n96b8.istio-system      Kubernetes     SYNCED     SYNCED     SYNCED     NOT SENT     NOT SENT     istiod-8495d444bb-w2j4p     1.14.1
  istio-ingressgateway-6668f9548d-5s2kp.istio-system     Kubernetes     SYNCED     SYNCED     SYNCED     SYNCED       NOT SENT     istiod-8495d444bb-w2j4p     1.14.1
  productpage-v1-7795568889-j4dhl.default                Kubernetes     SYNCED     SYNCED     SYNCED     SYNCED       NOT SENT     istiod-8495d444bb-w2j4p     1.14.1
  ratings-v1-754f9c4975-d2jzq.default                    Kubernetes     SYNCED     SYNCED     SYNCED     SYNCED       NOT SENT     istiod-8495d444bb-w2j4p     1.14.1
  reviews-v1-55b668fc65-ffnxv.default                    Kubernetes     SYNCED     SYNCED     SYNCED     SYNCED       NOT SENT     istiod-8495d444bb-w2j4p     1.14.1
  reviews-v2-858f99c99-fwtw2.default                     Kubernetes     SYNCED     SYNCED     SYNCED     SYNCED       NOT SENT     istiod-8495d444bb-w2j4p     1.14.1
  reviews-v3-7886dd86b9-jw8x4.default                    Kubernetes     SYNCED     SYNCED     SYNCED     SYNCED       NOT SENT     istiod-8495d444bb-w2j4p     1.14.1
  ```
- ![istio-prox-status](https://user-images.githubusercontent.com/63401132/178964112-b25a6981-073e-4cbe-8084-c44a529f3cb9.jpeg)

### istioctl proxy-config
- istio-proxy들의 현 config 상태를 확인해볼 수 있는 명령어
  - ````
    (gke_penguin-fly-dev_asia-northeast3-a_istio-study-cluster:default)➜  istio-1.14.1 istioctl proxy-config cluster productpage-v1-7795568889-j4dhl.default
    SERVICE FQDN                                            PORT      SUBSET     DIRECTION     TYPE             DESTINATION RULE
    9080      -          inbound       ORIGINAL_DST     
    BlackHoleCluster                                        -         -          -             STATIC           
    InboundPassthroughClusterIpv4                           -         -          -             ORIGINAL_DST     
    PassthroughCluster                                      -         -          -             ORIGINAL_DST     
    agent                                                   -         -          -             STATIC           
    default-http-backend.kube-system.svc.cluster.local      80        -          outbound      EDS              
    details.default.svc.cluster.local                       9080      -          outbound      EDS              
    grafana.istio-system.svc.cluster.local                  3000      -          outbound      EDS              
    istio-egressgateway.istio-system.svc.cluster.local      80        -          outbound      EDS              
    istio-egressgateway.istio-system.svc.cluster.local      443       -          outbound      EDS              
    istio-ingressgateway.istio-system.svc.cluster.local     80        -          outbound      EDS              
    istio-ingressgateway.istio-system.svc.cluster.local     443       -          outbound      EDS              
    istio-ingressgateway.istio-system.svc.cluster.local     15021     -          outbound      EDS              
    istio-ingressgateway.istio-system.svc.cluster.local     15443     -          outbound      EDS              
    istio-ingressgateway.istio-system.svc.cluster.local     31400     -          outbound      EDS              
    istiod.istio-system.svc.cluster.local                   443       -          outbound      EDS              
    istiod.istio-system.svc.cluster.local                   15010     -          outbound      EDS              
    istiod.istio-system.svc.cluster.local                   15012     -          outbound      EDS              
    istiod.istio-system.svc.cluster.local                   15014     -          outbound      EDS              
    jaeger-collector.istio-system.svc.cluster.local         9411      -          outbound      EDS              
    jaeger-collector.istio-system.svc.cluster.local         14250     -          outbound      EDS              
    jaeger-collector.istio-system.svc.cluster.local         14268     -          outbound      EDS              
    kiali.istio-system.svc.cluster.local                    9090      -          outbound      EDS              
    kiali.istio-system.svc.cluster.local                    20001     -          outbound      EDS              
    kube-dns.kube-system.svc.cluster.local                  53        -          outbound      EDS              
    kubernetes.default.svc.cluster.local                    443       -          outbound      EDS              
    metrics-server.kube-system.svc.cluster.local            443       -          outbound      EDS              
    productpage.default.svc.cluster.local                   9080      -          outbound      EDS              
    prometheus.istio-system.svc.cluster.local               9090      -          outbound      EDS              
    prometheus_stats                                        -         -          -             STATIC           
    ratings.default.svc.cluster.local                       9080      -          outbound      EDS              
    reviews.default.svc.cluster.local                       9080      -          outbound      EDS              
    sds-grpc                                                -         -          -             STATIC           
    tracing.istio-system.svc.cluster.local                  80        -          outbound      EDS              
    tracing.istio-system.svc.cluster.local                  16685     -          outbound      EDS              
    xds-grpc                                                -         -          -             STATIC           
    zipkin                                                  -         -          -             STRICT_DNS       
    zipkin.istio-system.svc.cluster.local                   9411      -          outbound      EDS 
    ```
- istioctl proxy-config --help
  - ````
    (gke_penguin-fly-dev_asia-northeast3-a_istio-study-cluster:default)➜  istio-1.14.1 istioctl proxy-config --help                                         
    A group of commands used to retrieve information about proxy configuration from the Envoy config dump
  
    Usage:
    istioctl proxy-config [command]
  
    Aliases:
    proxy-config, pc
  
    Examples:
    # Retrieve information about proxy configuration from an Envoy instance.
    istioctl proxy-config <clusters|listeners|routes|endpoints|bootstrap|log|secret> <pod-name[.namespace]>
  
    Available Commands:
    all            Retrieves all configuration for the Envoy in the specified pod
    bootstrap      Retrieves bootstrap configuration for the Envoy in the specified pod
    cluster        Retrieves cluster configuration for the Envoy in the specified pod
    endpoint       Retrieves endpoint configuration for the Envoy in the specified pod
    listener       Retrieves listener configuration for the Envoy in the specified pod
    log            (experimental) Retrieves logging levels of the Envoy in the specified pod
    rootca-compare Compare ROOTCA values for the two given pods
    route          Retrieves route configuration for the Envoy in the specified pod
    secret         Retrieves secret configuration for the Envoy in the specified pod
  
    Flags:
    -h, --help            help for proxy-config
    -o, --output string   Output format: one of json|yaml|short (default "short")
  
    Global Flags:
    --context string          The name of the kubeconfig context to use
    -i, --istioNamespace string   Istio system namespace (default "istio-system")
    -c, --kubeconfig string       Kubernetes configuration file
    -n, --namespace string        Config namespace
    --vklog Level             number for the log level verbosity. Like -v flag. ex: --vklog=9
  
    Use "istioctl proxy-config [command] --help" for more information about a command.
    ```
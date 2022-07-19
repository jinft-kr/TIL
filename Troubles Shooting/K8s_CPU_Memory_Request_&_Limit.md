### [ Issue ]
- 프론트엔드 서비스를 새로 배포 하였는데, 정상적으로 deploy 가 뜨지 않았다.
  - ![new_service_deploy](https://user-images.githubusercontent.com/63401132/177173205-607b12df-eb38-48b6-a7a7-b5fb25359e9b.jpeg)
- cpu, memory resource가 limit보다 높은 문제였다.
- 내가 직접 cpu, memory resource limit 를 설정한 것 없는데 뭐가 문제일까..

---

### [ Problem Solution Approach ]
- 프론트엔드 서비스를 새로 배포 하였는데, 정상적으로 deploy 가 뜨지 않았다.
    - ![new_service_deploy](https://user-images.githubusercontent.com/63401132/177173205-607b12df-eb38-48b6-a7a7-b5fb25359e9b.jpeg)
- describe 명령어를 이용하여 상세 조회를 해보았다.
  - ![describe_deploy](https://user-images.githubusercontent.com/63401132/177176021-c2722241-8671-4cd9-b6e5-fd9ba9e1ce6d.jpeg)
- replica가 생성되는데 실패했다는 걸 알 수 있었다.
- 왜 실패 했는지 알아보기 위해 event log를 찾아봤다.
  - ![event_log](https://user-images.githubusercontent.com/63401132/177176199-176cd723-2b22-4b42-8bcb-aec9644bc619.jpeg)
- cpu, memory resource가 limit보다 높은 문제였다.
- 내가 직접 cpu, memory resource limit 를 설정한 것 없는데 뭐가 문제일까..
- 로그 내용을 그대로 google에 검색해봤다.
  - ![google_limit_range](https://user-images.githubusercontent.com/63401132/177173468-f19fb3b7-651f-42a8-82ca-3d920d3a0062.jpeg)
- 제일 먼저 나오는 링크를 들어가보니 k8s resource 중 limit range라는 것이 있었다.
  - limit range : namespace에서 컨테이너와 파드가 사용하는 CPU 리소스의 최솟값과 최댓값을 설정하는 리소스
    - [네임스페이스에 대한 CPU의 최소 및 최대 제약 조건 구성](https://kubernetes.io/ko/docs/tasks/administer-cluster/manage-resources/cpu-constraint-namespace/)
- 혹시 누군가가 기존에 설정해놓은 내용이 있는지 조회해봤다.
  - ![get_limit_range](https://user-images.githubusercontent.com/63401132/177173891-7e6b7c0c-f8a8-4a35-ba52-0be3e9b3e4da.png)
- 아래와 같이 정의가 되어 있었다.
  - ```
    apiVersion: v1
    kind: LimitRange
    metadata:
      annotations:
    name: defaultlimit
    namespace: ${NAMESPACE}
    spec:
    limits:
    - default:
        cpu: 100m
        memory: 128Mi
      defaultRequest:
        cpu: 100m
        memory: 128Mi
      max:
        cpu: 200m
        memory: 256Mi
      type: Container
    ```
- 리소스를 2배로 늘려봤다.
  - ```
    apiVersion: v1
    kind: LimitRange
    metadata:
      annotations:
    name: defaultlimit
    namespace: ${NAMESPACE}
    spec:
    limits:
    - default:
        cpu: 200m
        memory: 256Mi
      defaultRequest:
        cpu: 200m
        memory: 256Mi
      max:
        cpu: 400m
        memory: 512Mi
      type: Container
    ```
- 리소스를 2배로 늘렸음에도 replica가 생성되지 않아 deploy가 정상적인 상태가 아니였다.
  - ![new_service_deploy](https://user-images.githubusercontent.com/63401132/177173205-607b12df-eb38-48b6-a7a7-b5fb25359e9b.jpeg)
- 기존에 있는 home-webapp-reactjs deploy를 삭제하고, 다시 새로운 deploy를 띄워봤다.
  - ![delete_deployment](https://user-images.githubusercontent.com/63401132/177174866-cdcb809e-58b0-431a-b1ba-2ade3e8bf963.jpeg)
- 오…. 안뜬다~
  - ![get_deploy](https://user-images.githubusercontent.com/63401132/177175035-95085b35-06bc-4068-a3c4-220d992c84b3.jpeg)
- limit range resource를 지워버렸다.
  - ![delete_limit_range](https://user-images.githubusercontent.com/63401132/177175253-26b90c5b-f59e-4e15-9b83-fa7c9cb4be92.jpeg)
- 정상적으로 잘 뜬다.
  - ![success_deploy](https://user-images.githubusercontent.com/63401132/177175480-1b09fcd0-057b-47b2-aa07-722280ca9258.jpeg)
- 기존 리소스를 다 삭제하고, cpu/memory 리소스 제한을 두지 않았으며 새로 띄웠으니 당연히 뜰 것이다.^^
- 따라서 쿠버네티스에서 CPU/Memory Request & limit를 어떻게 다루는지와
- limit range resource를 2배로 늘었음에도 불구하고 새로 배포한 deploy가 왜 안떴는지를 공부해보기로 했다.

---

### [Concept]
- 우선 CPU/Memory 기본 단위
  - CPU 기본 단위 : ms
  - 1vCPUs = 약 1000ms
  - Memory 기본 단위: 보통 Mi 단위

- K8s Default CPU/Memory Request & Limit
- 우선 쿠버네티스에서 limit range를 지정해주지 않았을 때 자동으로 설정해주는 기본 값은
  - [네임스페이스에 대한 기본 CPU 요청량과 상한 구성](https://kubernetes.io/ko/docs/tasks/administer-cluster/manage-resources/cpu-default-namespace/)
  - [네임스페이스에 대한 기본 메모리 요청량과 상한 구성](https://kubernetes.io/ko/docs/tasks/administer-cluster/manage-resources/memory-default-namespace/)
  - ````
    resources:
      limits:
        cpu: 1
        memory: 512Mi
      requests:
        cpu: 0.5 # 500m
        memory: 256Mi
    ````
- request: 컨테이너가 생성될 때 요청하는 리소스 양
- limit: 컨테이너가 생성된 후에 실행되다가 리소스가 더 필요할 경우 더 사용할 수 있는 양
  - 예) CPU request를 500ms로 하고, limit을 1000ms로 하면 해당 컨테이너는 처음에 생성될때 500ms를 사용할 수 있다. 그런데, 시스템 성능에 의해서 더 필요하다면 CPU가 추가로 더 할당되어 최대 1000ms 까지 할당될 수 있다.

---

### [ home namespace에 Limit Range Resource를 2배로 늘렸을 때 새로운 deploy가 안 뜬 이유! ]

---

### [ 추가적으로 내 클러스터 내에 사용 가능한 resource monitoring ]
- 현재 sc-service cluster는 ec2-medium type 의 VM을 사용하고 있고 현재 7개가 띄워져있다.
  - ec2-medium spec: 1vCPUs / 4GiB
  - [GCPinstances.info](https://gcpinstances.doit-intl.com/?selected=e2-medium)
  - ![GKE_Node_Pool](https://user-images.githubusercontent.com/63401132/177172663-0408be2a-59f4-428d-b73d-0a1dc8c23661.png)
- 1코어 VM 7대 = 총 7000m를 사용할 수 있을 것 같지만, 이 자원을 모두 사용자 애플리케이션에 사용할 수 있는 것이 아니다.
- 쿠버네티스 클러스터를 유지하는 시스템 자원이나 또는 모니터링등에 자원이 소비되기 때문에 실제로 사용할 수 있는 자원의 양은 다음과 같다.
  - kubectl describe node
  - ![Cluster_Resource](https://user-images.githubusercontent.com/63401132/177172346-02d23b68-71ef-4f67-a394-8cc8bcda9e65.jpeg)
  - cluster의 총 사용 가능 resource는 CPU: 4100m / Memory: 3822Mi

---

### [ 구글에서 제안하는 Resource 전략 ]
- [Kubernetes best practices: Resource requests and limits](https://cloud.google.com/blog/products/containers-kubernetes/kubernetes-best-practices-resource-requests-and-limits)
  - 데이터 베이스 등 아주 무거운 애플리케이션이 아니면, 일반적인 경우에는 CPU request를 100m 이하로 사용하기를 권장한다.
  - 또한 세밀하게 클러스터를 운영하기 어려운 경우에는 request와 limit의 사이즈를 같게 하는 것을 권장한다.
  - limit이 request보다 클 경우 overcommitted 상태가 발생할 수 있는데, 이때 CPU가 throttle down 되면, 실제 필요한 CPU양 보다 작은 CPU양으로 줄어들기 때문에 성능저하가 발생할 수 있다.
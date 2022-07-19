### [ Issue ]

- 팀원분께서 갑자기 질문을 하셨다.
- 'Kubernetes에서 Istio Api를 지원하지 않는데, kubectl 명령어로 Istio Resource가 어떻게 관리 되는건가요?'
- 따라서 왜 `kubectl` 명령어로 `istio resource`가 관리가 되는지 정리 해보기로 했다.

### [ Problem Solution Approach ]

- 우선 [kubernetes api version](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.23/#-strong-api-groups-strong-) 은 docs에서 확인해볼 수 있다. 
- 또 다른 방법으로 현재 사용하는 `kubectl` `version` 에서 지원하는 `api` 를 알고 싶다면, `kubectl api-versions` 명령어를 통해서도 확인이 가능하다.
  - ![kubectl_api_versions](https://user-images.githubusercontent.com/63401132/176878027-d61e2f5b-df0d-4a6f-b962-0bebc9796a35.jpeg)
  - 사진에 있는 `api version` 이외에도 많이 있다.
- 자주 사용하는 `istio api` 의 예시로는 `networking.istio.io/v1beta1` 가 있는데, `kubectl apiVersion` 에서는 지원하지 않는다.
- 그런데 어떻게 `kubectl` 을 통해 해당 `apiVersion` 을 사용할 수 있을까?

- 정답은 [Istio Operator](https://istio.io/latest/docs/setup/install/operator/) 에 있다.
  - `Istio Operator` 를 설치하연, `Kubernetes CRD API` 를 통해 `kubernetes` 에서도 `istio` 의 변경 사항을 적용할 수 있게 한다.
  - ![Istio_Operator](https://user-images.githubusercontent.com/63401132/176880271-16b51ce4-9c12-42e7-9c62-3871b7b74369.jpeg)
  - [Istio Operator 구성요소](https://github.com/istio/istio.io/blob/35d301204168e659d93779a5ddf323e065536b80/archive/v1.4/operator.yaml)

- CustomReouceDefinition
```
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:
  name: istiocontrolplanes.install.istio.io
spec:
  group: install.istio.io
  names:
    kind: IstioControlPlane
    listKind: IstioControlPlaneList
    plural: istiocontrolplanes
    singular: istiocontrolplane
    shortNames:
    - icp
  scope: Namespaced
  subresources:
    status: {}
  validation:
    openAPIV3Schema:
      properties:
        apiVersion:
          description: 'APIVersion defines the versioned schema of this representation
            of an object. Servers should convert recognized schemas to the latest
            internal value, and may reject unrecognized values.
            More info: https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api-conventions.md#resources'
          type: string
        kind:
          description: 'Kind is a string value representing the REST resource this
            object represents. Servers may infer this from the endpoint the client
            submits requests to. Cannot be updated. In CamelCase.
            More info: https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api-conventions.md#types-kinds'
          type: string
        spec:
          description: 'Specification of the desired state of the istio control plane resource.
            More info: https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api-conventions.md#spec-and-status'
          type: object
        status:
          description: 'Status describes each of istio control plane component status at the current time.
            0 means NONE, 1 means UPDATING, 2 means HEALTHY, 3 means ERROR, 4 means RECONCILING.
            More info: https://github.com/istio/operator/blob/master/pkg/apis/istio/v1alpha2/v1alpha2.pb.html &
            https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api-conventions.md#spec-and-status'
          type: object
  versions:
  - name: v1alpha2
    served: true
    storage: true
```

- [Custom Resources](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/)
  - Custom Resurce는 Kubernetes API의 확장이다. Kubernetes에서 지원하지 않는 Custom한 Resource를 추가할 때 사용한다.
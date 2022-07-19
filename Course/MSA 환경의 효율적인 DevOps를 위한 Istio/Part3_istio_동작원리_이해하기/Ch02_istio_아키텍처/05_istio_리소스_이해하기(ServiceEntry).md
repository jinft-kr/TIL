### 05. istio 리소스 이해하기()

라우팅에 관련하지만 고급기능
기본적으로 서비스를 라우팅하는데 필요하진 않지만
외부 서비스를 라우팅할 때 서비스메쉬 내에 있는것 처럼 등록할 수 있음
엔트리를 서비스 메쉬 내에 등록할 때 쓴다.

### Service Entry
- 메쉬 내에 네트워크를 여태 자동으로 등록했지만
- 수동으로 등록하겠다!!!
- Istio 내에서 어플리케이션이 접촉하는 모든 앱에 대해 서비스 레지스트리의 일부로 노출시키고자하는 욕망 반영
  - user case
    - RDS, DB, Cache 같은걸 서비스 메쉬에 있는 것 처럼 가시성을 확보하고 싶을 때 사용하고
    - 외부 서비스에 대해서도 등록해서 사용하기도 함
- 외부리소스에 대한 설정 +
- 내부 Static 라우팅에 대한 설정도 포함 +
- 결국, 인보이 프록시가 마치 해당서비스가 서비스메쉬에 있는 것처럼 등록하기 위함
  - Use case
    - kiali에 unknown, path throw cluster라 해서 도메인이 메쉬내에 없다고 판단될 때 뜨는데
    - 도메인으로 등록하며 서비스 엔트리의 이름으로 뜸
- 서비스 레지스트리에 등록된 서비스는 결국 Istio에서 제공하는 트래픽 옵션들을 활용할 수 있음
  - retry, timeout, fault injection ...
    - virtualService, destinationrule을 사용할 수 있기 때문에
  - 각각의 sidecar들이 해당 서비스 엔트리를 보고 동작하기 때문에 client 단에서 조절할 수 있는 옵션들이 풍부해진다.
- 등록하지 않은 외부 서비스는 unknown으로 뜨게되는데 메트릭상에서 보면 굉장히 안예쁨
  - use case
    - SRE입장에서는 수많은 RDS, Cache서버를 등록하는게 어려울 수도 있음
- 서비스 레지스트리에 등록된 서비스는 결국 Istio에서 제공하는 트래픽 옵션들을 활용할 수 있음
- VirtualService, DestinationRule로 활용할 수 있게되고 외부서비스를 마치 서비스 메쉬 서비스처럼 다루기
  - 클러스터 밖에있는 서비스(에를 들어 Compute Engine)에 MTLS를 적용해야하는데 service entry의 destinationrule을 적용해서 손쉽게 MTLS를 적용할 수 있다.


### ServiceEntry 유의할 것
- Host 와일드카드 prefix 도메인도 가능
  - DNS 타입이고 endpoints 리스트가 없다면 DNS 서비스처럼 쓰인다고 생각.
  - 만약 쿠버네티스랑 hostname이 같다면 ServiceEntry는 그걸 기존 k8s 리소스에 데코레이터처럼 동작하게됨

### ServiceEntry 유의할 것2
- 기본적으로 서비스엔트리는 모든 서비스/네임스페이스에 내보내짐
- exportTo 옵션으로 경계설정도 중요(현재 네임스페이스 제한)

### 적용 후 확인하는 꿀팀
- istioctl pc clusters {pod_name}.{namespace} --port {port} --direction outbound -o json
- Example)
  - istiocatl pc clusters test-123123123.adsad.test --port 11211 --direction outbound -o json
- pc: proxy config의 약자

### [ServiceEntry](https://istio.io/latest/docs/reference/config/networking/service-entry/)
- 
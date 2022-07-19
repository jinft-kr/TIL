### 01. istio 리소스를 이용해 트래픽 라우팅
- [Gateway](https://istio.io/latest/docs/reference/config/networking/gateway/)
    - 프로토콜 및 Gateway Port 설정
    - 받고자하는 hosts 등록 가능
    - TLS 관련 옵션 추가 가능
    - 실행하고자 하는 워크로드 proxy에 설정도 가능

- [VirtualService](https://istio.io/latest/docs/reference/config/networking/virtual-service/)
    - spec.hosts 에 처리할 트래픽 hosts 등록
    - L7 패쓰별 라우팅
    - 목적지에 대한 정책이 들어가는 곳이기 때문에 매우 중요한 리소스
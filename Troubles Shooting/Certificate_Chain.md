### [ Issue ]

- 기존 개발환경에서는 cert-manager를 이용하여 인증서를 발급 받아 매번 갱신하면서 사용했었다.
- 하지만 인증서에 대한 관리 포인트를 줄이기 위해서, 비용이 적은 Sectigo에서 도메인 인증서를 발급 받아 사용하기로 하였다.
- 그러나 인증 받은 `*.sample.com`(예시) 도메인에서 https 통신시, `chrome` 과 `IOS` 는 정상적으로 인증서가 인증되는데, `Android` 에서만 문제가 발생하였다.
- ```
  //터미널 에러
  graphql codegen Unable to verify first certificate
  ```
- ```
  //안드로이드 에러
  TypeError: Network request failed
  ```
- ![Android_Network_Request_fail_Error](https://user-images.githubusercontent.com/63401132/175935653-ffbd13a9-1545-4fb5-9dc3-2c0bfc545437.png)

### [ Problem Solution Approach ]

- SSL Labs(https://www.ssllabs.com/)에 원하는 서비스 도메인을 검색하여 분석한 결과, Android에서는 신뢰하지 않는 인증서라는 것을 알게 되었다.
  - SSL Server Test (Powered by Qualys SSL Labs) : ssl 분석 결과 제공 사이트
- 기존에 kubernetes istio-ingress 에 Secret Certificate로 `PositiveSSL Wildecard Certificate` 만 등록되어 있었다.
  - ```
    apiVersion: v1
    kind: Secret
    metadata:
    name: star-sample-com-tls
    namespace: istio-ingress
    type: kubernetes.io/tls
    data:
    tls.crt: ${STAR_sample_com.crt}
    tls.key: ${server.key}
    ```
- `PositiveSSL Wildcard Certificate` 만 등록되어 있던 것을, `Certificate Chain`으로 모든 인증서를 등록하였다.
  - PositiveSSL Wildcard Certificate - STAR_sample_com.crt
  - Intermediate CA Certificate - SectigoRSADomainValidationSecureServerCA.crt
  - Intermediate CA Certificate - USERTrustRSAAAACA.crt
  - Root CA Certificate - AAACertificateServices.crt
  - 
    ```
    apiVersion: v1
    kind: Secret
    metadata:
    name: star-sample-com-tls-chain
    namespace: istio-ingress
    type: kubernetes.io/tls
    data:
    tls.crt: ${STAR_sample_com_tls_chain.crt}
    tls.key: ${server.key}
    ```
- `SSL Labs` 를 통해 SSL 분석을 해본 결과 `Trusted List` 에 `Android` 가 추가되었다.
  - ![Certificate_Trusted](https://user-images.githubusercontent.com/63401132/175940835-8f5497ef-193a-4ce5-bcf4-b4ced50fe728.jpeg)


### [ Web & IOS에서는 Widecard Certificate만 등록해도 문제가 없었던 이유 ]

- [iOS allowed invalid certificate while android did not](https://stackoverflow.com/questions/33222877/ios-allowed-invalid-certificate-while-android-did-not) : stackoverflow에 동일한 issue 내용
- `일부 Browse`r 또는 `IOS` 에서는 `AIA chasing` 기능을 통해 유효하지 않은 인증서의 경우에 서버에서 인증서를 제공하지 않으면, 중간 인증서를 제공하는지 확인해 주는 기능이 있다. 
- 하지만 `Android` 에서는 기능을 지원하지 않는다.
- [AIA Fetching technology for restoring chain of certificates](https://www.leaderssl.com/es/articles/377-aia-fetching-technology-for-restoring-chain-of-certificates)
- ![star_sample_com_Certificate_Chain](https://user-images.githubusercontent.com/63401132/175939546-76ae1eb6-cc38-42c3-ac1a-389c26a16f11.png)

### [ Concept - Certificate ]
- `CA(Certificat Authority, 인증 기관)` : 인증서 발급 회사
- 인증서 종류
  - `Root CA Certificate` : AAACertificateServices.crt
    - CA에서 자체 서명(self-signed)한 인증서로 공개키 기반 암호화를 사용
  - `Intermediate CA Certificate` : USERTrustRSAAAACA.crt, SectigoRSADomainValidationSecureServerCA.crt
    - `Root Certificate` 와 `SSL 인증서` 사이에 구분을 만들어 위험을 완화하도록 설계된 인증서
    - `Root Certificate` 가 가장 많은 권한을 갖고 보호되어야 하기 때문에 `Root Certificate` 가 손상될 경우를 대비
  - `Leaf Certificate` : STAR_sample_com.crt
    - 사용자가 구입하는 `SSL Certificate`
  - `Certificate Chain` :  인증서를 신뢰할 수 있도록 서명을 하며 만들어진 체인
    - `Leaf Certificate` - `Intermediate Certificate` - `Root Certificate` 순서대로 작성해야 함
  - ![Certificate_chain](https://user-images.githubusercontent.com/63401132/175939735-f6fb1245-c430-4129-9fac-18c537d35d8e.png)

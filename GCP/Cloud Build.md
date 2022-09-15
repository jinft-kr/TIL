- Google Cloud Platform의 Cloud Build 서비스를 사용하는 방법에 대해 정리한 글입니다

---

### CI/CD
- CI
  - Continuous Integration
  - 테스트와 빌드를 자동으로 진행하는 프로세스
- CD
  - Continuous Deploy, Continuous Delivery
  - 배포 자동화
- 전략에 따라 CI, CD를 하나의 도구에서 할 수 있고, CI 따로 CD 따로 설정할 수 있음
- 대표적인 도구
  - Jenkins
    - Travis CI
    - CircleCI
    - Github Action
    - Buddy works
    - Google Cloud Build
    - 이외에도 정말 다양한 도구들이 있음
    - 여기선 Google의 Cloud Build에 대해 설명함

### Cloud Build
- Cloud Build는 Google Cloud Platform의 인프라에서 빌드를 실행하는 서비스
  - 다양한 저장소, 클라우드 스토리지 공간에서 소스 코드 가져오고, 사양에 맞게 선택
- 특징
  - Docker 지원
  - 매일 120분의 무료 빌드와 최대 10회 동시 빌드 지원
  - 빌드 과정 모니터링 제공
  - 컨테이너 이미지의 패키지 취약점 자동으로 파악
  - 로컬 또는 클라우드에서 빌드(로컬에서 테스트 가능)
  - 다른 도구들과 비교는 사용할 상황에 따라 다르고, 비용이나 필요한 기능을 고려하면 좋음
  - Cloud Build는 Google Cloud Platform에서 사용하기에 간단하고, 가벼운 어플리케이션 구축시 사용하면 좋아서 선택함
- 비용
  - 시기에 따라 달라질 수 있으니, 문서를 참고하면 좋음
  - (20년 5월 16일 기준) 매일 120분의 무료 빌드 시간이 제공 중(n1-standard-1 옵션)
  - 빌드가 큐에 있는 동안엔 빌드 시간이 계산되지 않음
  - 머신 유형(n1-standard-1)은 $0.003/빌드 시간(분)
  - 머신 유형(n1-highcpu-8)은 $0.016/빌드 시간(분)
  - 머신 유형(n1-highcpu-32)은 $0.064/빌드 시간(분)

  
### Build Cofig(빌드 구성) 단계
- Build Config 파일을 작성함
- 이 파일에는 Unit Test, Docker 이미지 빌드, 패키징 등이 가능함
- Docker, gradle, maven, bazel 같은 빌드 도구로 artifact 생성 가능
- cloudbuild.yaml
- yaml 파일 또는 json으로 작성 가능
- 여러 Build Steps(빌드 단계를 묶어서 하나의 파일에 저장함
- 각 Build Step은 Docker 컨테이너에서 실행됨
- Build Step에서 사용하는 Builder 이미지는 Cloud Build에서 제공하는 이미지와 커뮤니티에서 제공한 이미지, 커스텀 이미지 등이 있음
- 참고로 컨테이너 이미지를 Cloud builder라고 부름
- Cloud Build 제공한 이미지 : https://github.com/GoogleCloudPlatform/cloud-builders
  - bazel, curl, docker, gcloud, git, go, gsutil, javac, kubectl, mvn, npm, wget, yarn 등
- 커뮤니티에서 제공한 이미지 : https://github.com/GoogleCloudPlatform/cloud-builders-community
  - airflow, ansible, awscli, bq, composer, dataflow, docker-compose, helm, hugo, make 등
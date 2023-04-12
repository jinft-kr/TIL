GKE(Google Kubernetes Engine)에서는 OAuth Scope를 사용하여 서비스 계정이 GCP 리소스에 대한 권한을 부여합니다.

- OAuth Scope는 GKE 클러스터를 생성할 때 지정하며, 다음과 같은 역할이 있습니다.

1. Read-only scope: GKE 클러스터 내에서 특정 리소스에 대한 읽기 권한을 허용합니다. 예를 들어 "https://www.googleapis.com/auth/compute.readonly" OAuth Scope는 Compute Engine에서 인스턴스를 조회할 수 있는 권한을 부여합니다.
2. Write scope: GKE 클러스터 내에서 특정 리소스에 대한 쓰기 권한을 허용합니다. 예를 들어 "https://www.googleapis.com/auth/compute" OAuth Scope는 Compute Engine에서 인스턴스를 생성, 수정, 삭제할 수 있는 권한을 부여합니다.
3. Full control scope: GKE 클러스터 내에서 모든 리소스에 대한 쓰기 권한을 허용합니다. 예를 들어 "https://www.googleapis.com/auth/cloud-platform" OAuth Scope는 GCP 전체에 대한 권한을 부여합니다.

- OAuth Scope는 클러스터 노드에 부여되는 서비스 계정의 권한을 제어합니다. 따라서 클러스터 내에서 특정 작업을 수행하려면 해당 작업에 필요한 OAuth Scope를 부여한 서비스 계정을 사용해야 합니다.

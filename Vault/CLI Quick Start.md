### [CLI Quick Start](https://learn.hashicorp.com/collections/vault/getting-started)

### What is Vault
- Vault 목적 : 인프라(예: 데이터베이스 자격 증명, 암호, API 키)에서 비밀을 관리하고 보호하는 것

### Install Vault In MacOS
- `Homebrew` 의 패키지 저장소 `HashiCorp` 탭 설치
  - `brew tap hashicorp/tap`
- `hashicorp/tap/vault` 설치
  - `brew install hashicorp/tap/vault`
- 최신 버전으로 업데이트 하는 경우 다음 명령어 실행
  - `brew upgrade hashicorp/tap/vault`
- 설치 확인
  - `vault` 
  - ```
    ➜  vault
    Usage: vault <command> [args]
    
    Common commands:
        read        Read data and retrieves secrets
        write       Write data, configuration, and secrets
        delete      Delete secrets and configuration
        list        List data or secrets
        login       Authenticate locally
        agent       Start a Vault agent
        server      Start a Vault server
        status      Print seal and HA status
        unwrap      Unwrap a wrapped secret
    
    Other commands:
        audit                Interact with audit devices
        auth                 Interact with auth methods
        debug                Runs the debug command
        kv                   Interact with Vault's Key-Value storage
        lease                Interact with leases
        monitor              Stream log messages from a Vault server
        namespace            Interact with namespaces
        operator             Perform operator-specific tasks
        path-help            Retrieve API help for paths
        plugin               Interact with Vault plugins and catalog
        policy               Interact with policies
        print                Prints runtime configurations
        secrets              Interact with secrets engines
        ssh                  Initiate an SSH session
        token                Interact with tokens
        version-history      Prints the version history of the target Vault server    
    ```
    
### Starting the Server
- Vault는 클라이언트/서버 응용프로그램으로 작동한다. 
- Vault 서버는 데이터 저장소 및 백엔드와 상호 작용하는 Vault 아키텍처의 유일한 부분이다. 
- Vault CLI를 통해 수행되는 모든 작업은 TLS 연결을 통해 서버와 상호 작용한다.

- Vault 개발 서버 시작
  - 개발 서버는 사전 구성된 내장 서버로 안전하지는 않지만 로컬에서 Vault를 사용하는 데 유용하다.
  - ```
    ➜  vault server -dev
    ==> Vault server configuration:
    
                 Api Address: http://127.0.0.1:8200
                         Cgo: disabled
             Cluster Address: https://127.0.0.1:8201
                  Go Version: go1.17.12
                  Listener 1: tcp (addr: "127.0.0.1:8200", cluster address: "127.0.0.1:8201", max_request_duration: "1m30s", max_request_size: "33554432", tls: "disabled")
                   Log Level: info
                       Mlock: supported: false, enabled: false
               Recovery Mode: false
                     Storage: inmem
                     Version: Vault v1.11.2, built 2022-07-29T09:48:47Z
                 Version Sha: 3a8aa12eba357ed2de3192b15c99c717afdeb2b5
    
    ==> Vault server started! Log data will stream in below:
    ...
    WARNING! dev mode is enabled! In this mode, Vault runs entirely in-memory
    and starts unsealed with a single unseal key. The root token is already
    authenticated to the CLI, so you can immediately begin using Vault.
    
    You may need to set the following environment variable:
    
        $ export VAULT_ADDR='http://127.0.0.1:8200'
    
    The unseal key and root token are displayed below in case you want to
    seal/unseal the Vault or re-authenticate.
    
    Unseal Key: ${Unseal Key}
    Root Token: ${Root Token}
    
    Development mode should NOT be used in production installations!
    ```
  - Unseal Key 및 Root Token 값이 출력된다.

  - 새 터미널을 연다.
  - Vault 클라이언트가 개발 서버와 통신하도록 한다.
    - `export VAULT_ADDR='http://127.0.0.1:8200'`
    - Vault CLI는 환경 변수 VAULT_ADDR를 사용하여 요청을 보낼 Vault 서버를 결정 한다.
  - unseal key 는 어딘가에 저장해둔.
  - 환경 변수 VAULT_TOKEN 값을 터미널 출력에 표시되는 생성된 루트 토큰 값으로 설정 한다.
    - `export VAULT_TOKEN=${Root Token}`

- 서버가 실행 중인지 확인
  - `vault status`
  - ```
    ➜ vault status     
    Key             Value
    ---             -----
    Seal Type       shamir
    Initialized     true
    Sealed          false
    Total Shares    1
    Threshold       1
    Version         1.11.2
    Build Date      2022-07-29T09:48:47Z
    Storage Type    inmem
    Cluster Name    vault-cluster-320d18eb
    Cluster ID      84de04f5-2c1b-af7a-171a-cb25572513d3
    HA Enabled      false
    ```
    
### Your First Secret
- Key/Value secrets engine
  - dev 모드 에서 Vault를 실행할 때 경로에서 Key/Value v2 secrets engine이 secret/ 경로에 활성화된다.
  - Key/Value secrets engine 은 Vault용으로 구성된 물리적 저장소 내에 임의의 비밀을 저장하는 데 사용되는 일반 키-값 저장소이다.
  - Vault에 기록된 Secret 은 암호화된 다음 백엔드 저장소에 기록된다. 따라서 백엔드 스토리지 메커니즘 은 암호화되지 않은 값을 절대 볼 수 없으며 Vault 없이 이를 해독할 수 없다.
  - Key/Value secrets engine 에는 버전 1, 2가 있는데, 차이점은 v2는 비밀 버전 관리를 제공하고 v1은 제공하지 않는다는 것이다.
  - command format : `vault kv <subcommand> [options] [args]`
    - destroy : 영구적으로 secret 제거
    - enable-versioning : 기존 K/V v1 저장소에 대한 버전관리 활성화
    - metadata: Vault의 K/V 저장소와 상호작용
    - patch : 기존 secret을 덮어쓰지 않고 secret 업데이트
    - rollback : 이전 버전의 secret으로 rollback
    - undelete : 삭제된 버전의 secret 복원
- 도움말 보는 법
  - `vault kv --help`
  - ```
    ➜  vault kv --help
    Usage: vault kv <subcommand> [options] [args]
  
      This command has subcommands for interacting with Vault's key-value
      store. Here are some simple examples, and more detailed examples are
      available in the subcommands or the documentation.
  
      Create or update the key named "foo" in the "secret" mount with the value
      "bar=baz":
  
          $ vault kv put -mount=secret foo bar=baz
  
      Read this value back:
  
          $ vault kv get -mount=secret foo
  
      Get metadata for the key:
  
          $ vault kv metadata get -mount=secret foo
            
      Get a specific version of the key:
  
          $ vault kv get -mount=secret -version=1 foo
  
      The deprecated path-like syntax can also be used, but this should be avoided 
      for KV v2, as the fact that it is not actually the full API path to 
      the secret (secret/data/foo) can cause confusion:   
    
          $ vault kv get secret/foo
  
      Please see the individual subcommand help for detailed usage information.
  
    Subcommands:
        delete               Deletes versions in the KV store
        destroy              Permanently removes one or more versions in the KV store
        enable-versioning    Turns on versioning for a KV store
        get                  Retrieves data from the KV store
        list                 List data or secrets
        metadata             Interact with Vault's Key-Value storage
        patch                Sets or updates data in the KV store without overwriting
        put                  Sets or updates data in the KV store
        rollback             Rolls back to a previous version of data
        undelete             Undeletes versions in the KV store
    ```
- Secret 생성
  - KV v2 secrets engine이 마운트되는 Mouth path에 K/V 추가
    - ```
    ➜  vault kv put -mount=secret hello foo=world
    == Secret Path ==
    secret/data/hello
  
    ======= Metadata =======
    Key                Value
    ---                -----
    created_time       2022-08-03T02:25:05.206996Z
    custom_metadata    <nil>
    deletion_time      n/a
    destroyed          false
    version            1
    ```
  - 여러 개의 K/V 추가하기
    - ```
      ➜  vault kv put -mount=secret hello foo=world excited=yes
      == Secret Path ==
      secret/data/hello
  
      ======= Metadata =======
      Key                Value
      ---                -----
      created_time       2022-08-03T02:26:02.520489Z
      custom_metadata    <nil>
      deletion_time      n/a
      destroyed          false
      version            2
      ```
- Secret 얻기
  - ```
    ➜  vault kv get -mount=secret hello
    == Secret Path ==
    secret/data/hello
  
    ======= Metadata =======
    Key                Value
    ---                -----
    created_time       2022-08-03T02:26:02.520489Z
    custom_metadata    <nil>
    deletion_time      n/a
    destroyed          false
    version            2
  
    ===== Data =====
    Key        Value
    ---        -----
    excited    yes
    foo        world
    ```
  - 특정 필드에 대해서만 값 얻기
    - ```
      ➜  vault kv get -mount=secret -field=excited hello
      yes
      ```
  - 특정 값 `json` 으로 출력하기
    - ```
      ➜ vault kv get -mount=secret -format=json hello | jq -r .data.data.excited
      yes
      ```
- secret 삭제
  - `vault kv delete` 명령어를 사용하여 secret 삭제
    - ```
      ➜  vault kv delete -mount=secret hello
      Success! Data deleted (if it existed) at: secret/data/hello
      ```
  - 삭제한 secret 출력해보기
    - ```
      ➜  vault kv get -mount=secret hello
      == Secret Path ==
      secret/data/hello
  
      ======= Metadata =======
      Key                Value
      ---                -----
      created_time       2022-08-03T02:26:02.520489Z
      custom_metadata    <nil>
      deletion_time      2022-08-03T02:30:03.023009Z
      destroyed          false
      version            2
      ```
    - `deletion_time` 삭제된 시간을 보여줌
    - `destroyed`가 `false`일 때, 삭제된 데이터를 복구할 수 있음
  - 삭제한 data 복구해보기
    - ```
      ➜  vault kv undelete -mount=secret -versions=2 hello
      Success! Data written to: secret/undelete/hello
      ➜  vault kv get -mount=secret hello
      == Secret Path ==
      secret/data/hello
  
      ======= Metadata =======
      Key                Value
      ---                -----
      created_time       2022-08-03T02:26:02.520489Z
      custom_metadata    <nil>
      deletion_time      n/a
      destroyed          false
      version            2
  
      ===== Data =====
      Key        Value
      ---        -----
      excited    yes
      foo        world
      ```
      
### Secrets Engines
- Secret Engine : secret을 저장, 생성 또는 암호화하는 Vault의 구성요소
  - K/V Secret Engine : 데이러를 저장하고 읽는 목적
  - 또 다른 Secret Engine : 다른 서비스에 연결하고, 요청시 자격 증명을 생성, 서비스로 암호화
- 없는 경로로 Secret 생성해보기
  - ```
    ➜  vault secrets enable -path=kv kv
    Error enabling: Error making API request.
  
    URL: POST http://127.0.0.1:8200/v1/sys/mounts/kv
    Code: 400. Errors:
  
    * path is already in use at kv/
    ```
    - 존재하지 않는 경로로 명령해서 오류 발생
- secret engine 활성화
  - 새로운 Secret Engine 활성화하기
    - ```
      ➜  vault secrets enable -path=kv kv
      Success! Enabled the kv secrets engine at: kv/
      ```
      - `vault secrets enable kv` 명령어와 동일
      - 활성화된 Secret Engine의 기본 경로는 기본적으로 Secret Engine의 이름이다.
  - Secret Engine 에 대한 정보 조해하기
    - ```
      ➜ vault secrets list
      Path          Type         Accessor              Description
      ----          ----         --------              -----------
      cubbyhole/    cubbyhole    cubbyhole_db02529c    per-token private secret storage
      identity/     identity     identity_19aa43da     identity store
      kv/           kv           kv_b003280f           n/a
      secret/       kv           kv_db0d2429           key/value secret storage
      sys/          system       system_b95ab918       system endpoints used for control, policy and debugging
      (gke_penguin-fly-dev_asia-northeast3-a_sc-service:default)➜  develop git:(main) ✗ vault kv put kv/hello target=world
      Success! Data written to: kv/hello
      ```
      - Vault 서버에 5개의 활성화된 Secret Engine이 있다.
      - Secret Engine의 유형, 해당 경로 및 설명이 없는 경우 `n/a`로 표시
  - Secret Engine에 Secret 생성
    - ```
      $ vault kv put kv/hello target=world
      Success! Data written to: kv/hello
      ```
  - `kv/hello` 경로에 저장된 Secret 조회
      - ```
        $ vault kv get kv/hello
  
        ===== Data =====
        Key       Value
        ---       -----
        target    world
        ```
  - `kv/my-secret` 경로에 secret 생성
    - ```
      ➜  vault kv put kv/my-secret value="s3c(eT"
      Success! Data written to: kv/my-secret
      ```
  - `kv/my-secret` secret 조회
    - ```
      ➜  vault kv get kv/my-secret
      ==== Data ====
      Key      Value
      ---      -----
      value    s3c(eT
      ```
  - `kv/my-secret` secret 삭제
    - ```
      ➜  vault kv delete kv/my-secret
      Success! Data deleted (if it existed) at: kv/my-secret
      ```
  - `kv` 경로에 저장된 key 조회
    - ```
      ➜  vault kv list kv/
      Keys
      ----
      hello
      ```
- secret engine 비활성화
  - ```
    ➜  vault secrets disable kv/
    Success! Disabled the secrets engine (if it existed) at: kv/
    ```
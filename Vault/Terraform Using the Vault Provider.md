### Terraform Using the Vault Provider
Terraform 에서 Vault 사용하는 법

### [Vault Provider](https://registry.terraform.io/providers/hashicorp/vault/latest/docs)
- Vault Provider Parameter
  - address 
    - (Required) Vault 서버의 URL. 환경변수 `VAULT_ADDR`를 통해 설정할 수도 있다.
  - add_address_to_env 
    - (Optional) default는 Fasle. `VAULT_ADDR`로 Terraform의 환경변수가 설정되는 경우 `true로 설정해야한다.
  - token : (Optional) 
    - 인증을 위해 Terraform에서 사용할 Vault token. 
    - 환경변수 `VAULT_TOKEN` 를 통해 설정할 수도 있다. 
    - 설정이되지 않은 경우 `~/.vault-token`에서 설정을 읽어 온다. 
    - `skip_child_token` 이 `true` 로 설정되있지 않으면 secret에 대한 노출을 제한하기 위해서 TTl과 함께 주어진 token의 자식인 새 토큰을 자체적으로 발행한다.
      - 자식 토큰을 생성하기 위해서는 Vault의 `auth/token/create` policy 가 있어야 한다. 
  - token_name 
    - (Optional) 자식 토큰을 생성할 때 Terraform에서 사용할 토큰 이름이다.
    - 환경변수 `VAULT_TOKEN_NAME` 을 통해 직접 설정할수도 있다.
  - ca_cert_file 
    - (Optional) Vault 서버에서 제공한 인증서의 유효성을 검사하는 데 사용할 로컬 디스크 파일 경로이다.
    - 환경변수 `VAULT_CACERT` 를 통해 직접 설정할 수도 있다.
  - ca_cert_dir 
    - (Optional) Vault 서버에서 제공한 인증서의 유효성을 검사하는 데 사용할 하나 이상의 인증서 파일이 포함된 로컬 디스크의 디렉토리 경로이다.
    - 환경변수 `VAULT_CAPATH` 를 통해 직접 설정할 수도 있다.
  - auth_login 
    - (Optional) Terraform이 사용할 토큰을 얻기위해 경로를 사용하여 인증을 시도하는 블록이다.
    - Terraform은 TTL을 적용하고 secret 노출을 제한하기 위해 `auth/token/create`를 사용하여 제한된 자식 토큰을 자체적으로 발행한다.
- `auth_login` block의 parameter
  - path : (Required) Vault 인증을 위한 로그인 경로
    - `auth/approle/login` 경로로 approle로 로그인 가능하다.
  - parameter : (Optional) 인증 백엔드에 인증할 때 보낼 K/V 의 Map

### Provider Dedugging
- TERRAFORM_VAULT_LOG_BODY : true로 설정하면 요청 및 응답 본문이 모두 기록된다.
- TERRAFORM_VAULT_LOG_REQUEST_BODY : true로 설정하면 요청 본문이 기록된다.
- TERRAFORM_VAULT_LOG_RESPONSE_BODY : ture로 설정하면 응답 본운이 모두 기록된다.

### Example
- basic
  - ```
    provider "vault" {
      # It is strongly recommended to configure this provider through the
      # environment variables described above, so that each user can have
      # separate credentials set in the environment.
      #
      # This will default to using $VAULT_ADDR
      # But can be set explicitly
      # address = "https://vault.example.net:8200"
    }
    
    resource "vault_generic_secret" "example" {
      path = "secret/foo"
    
      data_json = jsonencode(
        {
          "foo"   = "bar",
          "pizza" = "cheese"
        }
      )
    }
    ```
- `auth_login`의 example
  - ```
    variable login_username {}
    variable login_password {}
    
    provider "vault" {
      auth_login {
        path = "auth/userpass/login/${var.login_username}"
    
        parameters = {
          password = var.login_password
        }
      }
    }  
    ```
- `app_role`의 example
  - ```
    variable login_approle_role_id {}
    variable login_approle_secret_id {}
    
    provider "vault" {
      auth_login {
        path = "auth/approle/login"
    
        parameters = {
          role_id   = var.login_approle_role_id
          secret_id = var.login_approle_secret_id
        }
      }
    }
    ```
    
### 실무에 적용한 코드
```
provider "vault" {
  address = var.vault_address
  auth_login {
    path = "auth/userpass/login/${var.login_username}"

    parameters = {
      password = var.login_password
    }
  }
}

data "vault_generic_secret" "db_mysql_user" {
  path = "secret/db/mysql/user"
}

data "vault_generic_secret" "db_mysql_password" {
  path = "secret/db/mysql/password"
}

locals {
  db_mysql_user = data.vault_generic_secret.db_mysql_user.data[var.operation_env]
}

locals {
  db_mysql_password = data.vault_generic_secret.db_mysql_password.data[var.operation_env]
}
```

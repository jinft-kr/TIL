### General style and structure guidelines
- Follow a standard module structure.
- Adopt a naming convention.
- Use variables carefully.
- Expose outputs.
- Use data sources.
- Limit the use of custom scripts.
- Include helper scripts in a separate directory.
- Put static files in a separate directory.
- Protect stateful resources.
- Use built-in formatting.
- Limit the complexity of expressions.
- Use count for conditional values.
- Use for_each for iterated resources.
- Publish modules to a registry.

### Follow a standard module structure
- 모든 모듈의 시작은 `main.tf` 에서 시작한다.
- 모든 모듈에는 `README.md` 파일에 모듈에 대한 기본 문서를 포함한다.
- 예제는 `examples/` 디렉토리에 위치하며, 각각의 예제에 대해서는 세부적인 하위 디렉토리를 위치한다. 각각의 예제에 대한 자세한 설명은 `README.MD` 에 포함한다.
- `network.tf`, `instances.tf`, `loadbalancer.tf` 와 같은 파일명을 사용하여 리소스의 논리적 그룹을 만든다.
  - 모든 리소스 별로 개별 파일을 생성하지 않는다.
  - 공유 목적에 따라 리소스를 그룹화한다.
  - 예, `google_dns_managed_zone`, `google_dns_record_set` 은 `dns.tf`에 결합하여 사용한다.
  - 모듈의 루트 디렉토리에는 레포지토리에 대한 메타데이터(`README.md`, `CHANGELOG.md`)와 Terraform(`*.tf`) 파일만 포함한다.
- 추가 문서는 `docs/` 하위 디렉토리에 위치한다.

### Adopt a naming convention
- 여러 단어를 구분하기 위해서는 `_`을 사용한다.
  - 이 방법은 리소스 유형, 데이터 소스 유형 및 기타 미리 정의된 값에 대한 명명 규칙과의 일관성을 보장한다.
  - Recommended:
    - ```
      resource "google_compute_instance" "web_server" {
        name = "web-server"
      }
      ```
  - Not recommended:
    - ```
      resource "google_compute_instance" "web-server" {
        name = "web-server"
      }
      ```
- 어떤 유형에 대한 단일 리소스일 경우, 참조를 단순화 하려면 리소스 이름을 `main` 으로 한다.
  - 예, `some_google_resource.my_special_resource.id` 가 아닌 `some_google_resource.main.id` 이다.
- 동일한 유형의 리소스를 구분하려면 의미 있는 리소스 이름을 사용한다.
  - 예, `primary`, `secondary`
- 리소스 이름을 단일화하여라.
  - 이 부분에 대해서 개인적인 생각으로는, 간단한거보다 의미가 명확한 이름을 사용하는 것이 좋다.
- 리소스 이름에서 리소스 유형을 반복하지 마라.
  - Recommended:
    - ```
      resource "google_compute_global_address" "main" { ... }
      ```
  - Not recommended:
    - ```
      resource "google_compute_global_address" "main_global_address" { … }
      ```
      
### Use variables carefully
- `variables.tf` 에 모든 변수를 선언한다.
- 변수의 이름은 용도나 목적에 관련되어 작성한다.
  - 디스크 크기 또는 RAM 크기와 같은 숫자 값을 나타내는 변수는 다위로 이름을 지정해야한다.
    - 예: `ram_size_gb`
    - Google Cloud APIs에는 표준 단위가 없으므로 단위가 있는 변수 이름을 지정하면 예상되는 입력단위가 명확해진다.
  - 저장 단위의 경우 접두사(`kilo`, `mega`, `giga`)를 사용한다. 
- 변수에 설명을 작성한다.
- 변수타입을 명시한다.
- 적절한 경우에는 변수에 기본값을 명시한다.
  - 환경에 독립적인 값(예: 디스크 크기)이 있는 변수의 경우에는 기본값을 제공해라. 
  - 환경별로 값(예, `project_id`) 이 다른 변수의 경우에는 기본값을 제공하지 마라.
- 각 인스턴스 또는 환경에 대해 달라야 하는 값만 매개변수화한다. 

### Expose outputs
- 모든 출력은 `output.tf` 파일로 구성한다.
- 모든 출력에 대해 의미 있는 설명을 제공한다.
- 파일의 출석 설명을 `README.md` 에 문서화한다. [terraform-docs](https://github.com/terraform-docs/terraform-docs) 와 같은 도구를 사용하여 커밋 시 설명을 자동으로 생성한다.
- 루트 모듈이 참조하거나 공유해야 할 수 있는 모든 유용한 값을 출력한다. 특히 오픈 소스 또는 많이 사용되는 모듈의 경우 사용이 가능한 값들을 모두 output으로 노출한다.
- 입력 변수를 통해 직접 출력을 전달하지마라. 그렇게 하면 종속성 그래프에 적절하게 추가되지 않기 때문이다. 
  - Recommended:
    - ```
      output "name" {
        description = "Name of instance"
        value       = google_compute_instance.main.name
      }
      ```
  - Not recommended:
    - ```
      output "name" {
        description = "Name of instance"
        value       = var.name
      }
      ```
      
### Use data sources

### Limit the use of custom scripts
- 필요한 경우에만 스크립트를 사용해라. 스트립트를 통해 생성된 리소스 상태는 Terraform에서 설명하거나 관리하지 않는다.
  - 가능하면 사용자 정의 스크립트를 사용하지 말아라. Terraform 리소스가 원하는 동작을 지원하지 않는 경우에만 사용해라
  - 사용되는 모든 사용자 정의 스크립트에는 기존 및 이상적으로는 사용 중단 계획에 대해 명확하게 문서화된 이유가 있어야 한다.
  - Terraform에서 호출한 사용자 정의 스크립트를 `scripts/` elfprxhfldp sjgsmsek.

### Include helper scripts in a separate directory
- Terraform에서 호출하지 않는 helper script는 `helpers/` 디렉토리에 구성한다.
- `README.md` 에 helper script를 문서화한다.
- helper script가 인수를 허용하는 경우 인수 검사 및 `--help` 출력을 제공한다.

### Put static files in a separate directory
- Terraform이 참조하지만 실행하지 않는 정적파일(예, Compute Engine 인스턴스에 로드된 시작 스크립트)은 `files/` 디렉토리에 구성한다.
- HCL과 별도로 긴 HereDocs를 외부 파일에 배치한다. 
  - [file()](https://www.terraform.io/language/functions/file)
- Terraform [templatefile](https://www.terraform.io/language/functions/templatefile) 기능을 사용하여 읽어들인 파일의 경우 파일 확장자를 `.tftpl` 을 사용한다.
  - `templates/` 디렉토리에 있어야 한다.

### Protect stateful resources
- 데이터베이스와 같은 상태 저장 리소스의 경우 삭제 방지 가 활성화되어 있는지 확인한다.
  - ```
  resource "google_sql_database_instance" "main" {
    name = "primary-instance"
    settings {
      tier = "D0"
    }
  
    lifecycle {
      prevent_destroy = true
    }
  }
  ```
  
### Use built-in formatting
- 모든 Terraform 파일은 `terraform fmt` 의 표준을 준수해야 한다.

### Limit the complexity of expressions
- 표현식의 복잡성을 제한한다. 단일 표현식에 많은 함수가 필요한 경우 [local](https://www.terraform.io/language/values/locals) 을 사용하여 여러 표현식으로 분할하는 것이 좋다.
- 한 줄에 둘 이상의 삼항 연산을 사용하지 않는다. 대신에 여러 로컬 값을 사용하여 로직을 구축한다.

### Use `count` for conditional values
- 리소스를 조건에 따라 생성하려면 `count` 메타 인수를 사용해라
  - ```
    variable "readers" {
      description = "..."
      type        = list
      default     = []
    }

    resource "resource_type" "reference_name" {
      // Do not create this resource if the list of readers is empty.
      count = length(var.readers) == 0 ? 0 : 1
      ...
    }
    ```
- 예, `project_id` 에 대해 리소스 속성이 제공되는데 해당 리소스가 아직 존재하지 않는 경우 Terraform은 계획을 생성할 수 없다. 대신 Terraform은 `value of count cannot be computed` 오류를 보고한다.
  - 이러한 경우에는 `enable_x` 변수를 사용하여 조건부 로직을 만든다.

### Use `for_each` for iterated resources
- 입력 리소스를 기반으로 리서소의 여러 복사본을 생성하려면 `for_each` 메타 인수를 사용해라.

### Publish modules to a registry
- 재사용 가능한 모듀 : 재사용가능한 모듈은 모듈 레지스트리에 게시한다.
- 오픈 소스 모듈 : Terraform 레지스트리에 오픈 소스 모듈을 게시한다.
- 개인 모듈 : 개인 모듈은 개인 레지스트리에 게시한다.
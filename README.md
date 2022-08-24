# Terraform-Workshop

Terraform은 Resource를 선언적인 코드로 생성/관리 할 수 있도록 도와주는 도구 입니다. 이를 IaC(Infastructure as Code) 라고 부릅니다.
여기에서는 Terraform의 기본적인 개념들을 소개합니다.

* Configuration Language
* Commands (CLI)
* Providers
* Provisioners
* Modules
* State
 *Backends

## CONFIGURATION LANGUAGE

Terraform은 HCL(Hashicorp Configuration Language) 이라는 인프라에 대한 간결한 설명이 가능하도록 설계된 자체 구성 언어를 사용합니다.

#### Resources and Modules

Terraform 언어의 주요 목적은 Resource를 선언하는 것입니다. 리소스는 단일 인프라 개체를 정의 합니다.
리소스 그룹은 Module 이라는 더 큰 구성 단위를 만들 수 있습니다.

#### Arguments, Blocks, and Expressions

<pre>
<code>
resource"aws_vpc" "main"{
  cidr_block =var.base_cidr_block}
BLOCK_TYPE"BLOCK_LABEL" "BLOCK_LABEL"{
  // Block body
IDENTIFIER =EXPRESSION # Argument
}
</code>
</pre>

* Block은 다른 내용의 컨테이너이며 일반적으로 리소스와 같은 일종의 개체 구성을 나타냅니다. 블록은 Block Type을 가지며 0개 이상의 Block Label을 가질 수 있으며 여러 개의 인수와 중첩 된 블록을 포함하는 본문을 갖습니다.
* Argument는 이름에 값을 할당합니다. 그것들은 블록 안에 작성합니다.
* Expression은 문자 그대로 또는 다른 값을 참조하고 결합하여 값을 나타냅니다. 인수의 값으로 또는 다른 표현식 내에 나타납니다.

#### Code Organization

Terraform은 확장자가 .tf인 파일을 사용합니다. Json 기반일 경우 .tf.json을 사용합니다.

모듈은 하나의 디렉토리에 포함된 테라폼 파일의 모음입니다. 루트 모듈은 테라폼이 실행될 때 현재 작업의 디렉토리의 구성 파일에서 빌드 되며 이 모듈은 다른 디렉토리의 하위 모듈을 참조 할 수 있으며, 다른 모듈 등을 참조 할 수 있습니다.

#### Configuration Ordering

일반적으로 Terraform의 블록 순서는 중요하지 않습니다. 구성의 정의 관계에 따라 리소스를 자동으로 처리합니다.

#### Terraform CLI vs Providers

Terraform CLI는 Terraform 구성을 평가하고 적용하기 위한 엔진입니다. Terraform 언어 구문과 전체 구조를 정의하고 원격 인프라가 주어진 구성과 일치하도록 변경 순서를 조정합니다.

Terraform은 각각 일련의 리소스 유형을 정의하고 관리하는 Provider 라는 플러그인을 사용합니다. 대부분의 Provider는 특정 클라우드 또는 On-premise 인프라 서비스와 연결되어 있으므로 Terraform은 해당 서비스 내에서 인프라 개체를 관리할 수 있습니다

* Input Variables
* Providers
* Resources
* Output Values
* Local Values
* Data Sources
* Modules

### INPUT VARIABLES

Input Variable은 Terraform 모듈의 매개 변수 역할을 하며, 다른 모듈 간에 매개 변수를 공유 할 수 있습니다.

#### Declaring an Input Variable

Variable 블록을 사용하여 설정 합니다.

<pre>
<code>
variable"ami_id"{
  description ="The id of the machine image (AMI)."type        =string}
variable"availability_zone_names"{
  type    =list(string)
  default =[
    "us-west-1a",
    "us-west-1b",
  ]
}
variable"docker_ports"{
  type =list(object({
    internal =numberexternal =numberprotocol =string}))
  default =[
    {
      internal =8300external =8300protocol ="tcp"}
  ]
}
</code>
</pre>

Description 인수를 사용하여 각 값의 목적을 간단히 설명 할 수 있습니다.
Type 인수를 사용하면 변수 값으로 허용되는 타입을 제한 할 수 있습니다. 타입이 설정되지 않은 경우 모든 유형의 값이 허용됩니다.
Default 인수를 선언하면 해당 인수를 선언 하지 않았을때 정의된 값을 사용 합니다. 하지만 선언 하지 않으면 해당 값은 필수가 됩니다.

#### Using Input Variable Values

변수를 선언한 모듈 내에서 표현식 내에서 var.<NAME>으로 값에 액세스 할 수 있습니다.

<pre>
<code>  
resource "aws_instance" "example" {
  instance_type = "t2.micro"
  ami = var.ami_id
 }
</code>
</pre>

### PROVIDERS
  
자원(Resource)은 Terraform 언어의 기본 구성이지만 자원의 동작은 연관된 자원 유형에 의존하며 이러한 유형은 제공자(Provider)에 의해 정의됩니다.
각 제공자는 이름 지정된 자원 유형 세트를 제공하고 각 자원 유형에 대해 허용되는 인수, 내보내는 속성 및 해당 유형의 자원 변경 사항이 실제로 원격 API에 적용되는 방법을 정의합니다.
사용 가능한 대부분의 공급자는 하나의 클라우드 또는 On-premise 인프라 플랫폼에 해당하며 해당 플랫폼의 각 기능에 해당하는 리소스 유형을 제공합니다.
공급자는 일반적으로 엔드 포인트 URL, 리전, 인증 설정 등을 지정하기 위해 자체 구성이 피료합니다. 동일한 제공자에 속하는 모든 자원 유형은 동일한 구성을 공유하므로 모든 자원 선언에서 이 공통 정보를 반복할 필요가 없습니다.

#### Provider Configuration

Provider 블록을 사용하여 설정 합니다.

<pre>
<code>
provider"aws"{
  region =var.region}
</code>
</pre>

<pre>
<code>
provider"google"{
  region  =var.regionproject =var.project}
</code>
</pre>
  
  
  ```hcl
  
  
  ```

#### Initialization

새 제공자가 명시적으로 제공자 블록을 통해 또는 해당 제공자의 자원을 추가하여 구성에 추가될 때마다 Terraform은 제공자를 사용하기 전에 제공자를 초기화 해야 합니다. 초기화는 나중에 실행할수 있도록 공급자의 플러그인을 다운로드하여 설치합니다.

<pre>
<code>
terraform init
</code>
</pre>
  
### RESOURCE
  
Terraform 언어에서 리소스는 가장 중요한 요소입니다. 각 리소스 블록은 가상 네트워크, 컴퓨팅 인스턴스 또는 DNS 레코드와 같은 상위 구성 요소와 같은 하나 이상의 인프라 개체를 설명합니다.

Terraform 언어에서 리소스는 가장 중요한 요소입니다. 각 리소스 블록은 가상 네트워크, 컴퓨팅 인스턴스 또는 DNS 레코드와 같은 상위 구성 요소와 같은 하나 이상의 인프라 개체를 설명합니다.

#### Resource Syntax

아래는 aws ec2 instance를 잘 생성하는 가장 심플한 코드 입니다.

<pre>
<code>
resource"aws_instance" "sample"{
  ami           =var.ami_idinstance_type =var.instance_type}
</code>
</pre>

Resource 블록을 사용하여 설정 합니다. Aws_instance의 자원을 생성하며, sample의 이름을 가집니다.
블록 내의 ami instance_type은 aws_instance 자원 유형의 인수 입니다.

#### Resource Behavior

리소스 블록은 특정 인프라의 선언적 정의 입니다. 정의한 자원은 실제 인프라 개체를 타내지는 않습니다.
Terraform apply를 통하여 설정과 구성이 일치 하도록 실제 인프라 개체를 생성, 업데이트 및 삭제를 수행 합니다.
새 인프라 개체를 생성하면 State에 저장어 관리 됩니다.

#### Resource Dependencies

구성의 대부분의 리소스 간에는 특별한 관계가 없으며 Terraform은 관련이 없는 여러 리소스를 동시에 변경할 수 있습니다.
그러나 일부 자원은 다른 특정 자원 이후에 처리해야 합니다. 때로는 리소스 작동 방식 때문일 수도 있고 리소스 구성에 다른 리소스에서 생성 된 정보만 필요하기도 합니다.
대부분의 리소스 종속성은 자동으로 처리됩니다. Terraform은 리소스 블록 내의 표현식을 분석하여 다른 객체에 대한 참조를 찾 다음 해당 참조를 리소스를 생성, 업데이트 또는 파괴 할때 암시적 순서 요구 사항으로 취급 합니다. 다른 자원에 대한 행동 종속성이 있는 대부분의 자원도 해당 자원의 데이터를 참조하므로 일반적으로 자원간 종속성을 수동으로 지정할 필요는 없습니다.

그러나 일부 종속성은 구성에서 암시적으로 인식될 수 없습니다. 예를 들어 Terraform이 액세스 제어 정책을 관리하고 해당 정책이 있어야 하는 조치를 취해야 하는 경우 액세스 정책과 해당 정책이 종속된 리소스간에 숨겨진 종속성이 있습니다. 이러한 드문 경우에 depend_on 메타 인수는 명시적으로 종속성을 지정할 수 있습니다.

#### Meta-Arguments

Terraform CLI는 다음과 같은 메타 인수를 정의합니다. 이 인수는 모든 자원 유형과 함께 사용하여 자원의 동작을 변경할 수 있습니다.

* count: 개수에 따라 여러 자원을 작성
* depend_on: 숨겨진 종속성을 명시적으로 지정
* for_each: 맵 또는 문자열 세트에 따라 여러 자원을 작성
* provider: 기본이 아닌 공급자 구성을 선택
* lifecycle: 라이프 사이클 사용자 정의
* provisioner and connection: 리소스 생성 후 추가 작업을 수행

#### count

4개의 EC2 instance를 생성 합니다.

<pre>
<code>
resource "aws_instance" "server" {
  count = 4
  ami           = var.ami_id
  instance_type = var.instance_type
  tags = {
    Name = "Server ${count.index}"
  }
</code>
</pre>

<pre>
<code>
}
resource "aws_vpc" "this" {
  count = var.create_vpc ? 1 : 0
  cidr_block = var.vpc_cidr
}
</code>
</pre>

#### depends_on

EC2 instance를 생성하기 전, instance_profile을 생성해야 한다는것은 aws_insatance 안에 정의되어 있으므로 유추가 가능합니다. 하지만, aws_iam_role_policy는 Terraform이 유추해 낼 수 없으므로 선언해 주어야 합니다.

<pre>
<code>
resource "aws_iam_role" "example" {
  name               = var.name
  assume_role_policy = "..."
}
resource "aws_iam_instance_profile" "example" {
  role = aws_iam_role.example.name
}
resource "aws_iam_role_policy" "example" {
  name   = var.name
  role   = aws_iam_role.example.name
  policy = "..."
}
resource "aws_instance" "example" {
  ami           = var.ami_id
  instance_type = var.instance_type
  iam_instance_profile = aws_iam_instance_profile.example
  depends_on = [
    aws_iam_role_policy.example,
  ]
}
</code>
</pre>

#### for_each

Autoscaling Group에서 mixed_instances로 정의할 수 있는데 var.mixed_instances 배열로 n개의 값을 전달 할 수 있습니다.

<pre>
<code>
resource "aws_autoscaling_group" "worker" {
  name = var.name
  min_size = var.min
  max_size = var.max
  vpc_zone_identifier = var.subnet_ids
  mixed_instances_policy {
    launch_template {
      launch_template_specification {
        launch_template_id = aws_launch_template.worker.id
        version            = "$Latest"
      }
      override {
        instance_type = var.instance_type
      }
      dynamic "override" {
        for_each = var.mixed_instances
        content {
          instance_type = override.value
        }
      }
    }
  }
}
</code>
</pre>

<pre>
<code>
      override {
        instance_type = var.instance_type
      }
      override {
        instance_type = var.mixed_instances.0
      }
      override {
        instance_type = var.mixed_instances.1
      }
</code>
</pre>

#### lifecycle

리소스의 생명주기를 정의합니다.

<pre>
<code>
resource "aws_launch_configuration" "worker" {
  // ...
  lifecycle {
    create_before_destroy = true
  }
}
</code>
</pre>
  
create_before_destroy(bool) - 현재 업데이트 할 수 없는 리소스 인수를 변경해야하는 경우 Terraform은 기존 객체를 삭제한 다음 새로 구성된 인수로 새 대체 객체를 만듭니다.
prevent_destroy(bool) - 인수가 구성에 존재하는 한 자원과 관련된 인프라 개체를 파괴하는 계획을 Terraform이 오류와 함께 거부하게 합니다.
Ignore_changes(list of attribute names) - Terraform은 실제 인프라 개체의 현재 설정에서 차이를 감지하고 구성과 일치하도록 원격 개체를 업데이트를 시도 합니다. 하지만 외부에서 변경된 리소스를 변경 하고 싶지 않을때 사용될 수 있습니다.

### OUTPUT VALUES

Output Value는 Terraform 모듈의 반환 값과 유사하며 여러 용도로 사용됩니다.

#### Declaring an Output Value

Output 블록을 사용 하여 설정합니다.

<pre>
<code>
output "private_ip" {
  description = "The private IP address of the main server instance."
  value       = aws_instance.server.private_ip
  depends_on = [
    aws_security_group_rule.local_access,
  ]
}
output "db_password" {
  description = "The password for logging in to the database."
  value       = aws_db_instance.db.password
  sensitive   = true
}
</code>
</pre>

description 인수를 사용하여 각 값의 목적을 간단히 설명 할 수 있습니다.

value 인수는 결과가 사용자에게 리턴되는 표현식을 사용합니다.

sensitive 출력값을 마스킹 할 수 있습니다.

depends_on 출력 값은 모듈에서 데이터를 전달하는 수단 일 뿐이므로 일반적으로 종속성 그래프에서 다른 노드와의 관계에 대해 걱정할 필요가 없습니다. 그러나 상위 모듈이 하위 모듈 중 하나에서 내 보낸 출력 값에 액세스 할 때 해당 출력 값의 종속성을 통해 Terraform은 다른 모듈에 정의 된 리소스 간의 종속성을 올바르게 결정할 수 있습니다.

#### Accessing Child Module Outputs

상위 모듈에서 하위 모듈의 출력은 module.<MODULE NAME>.<OUTPUT NAME>로 표현할 수 있습니다. 예를 들어, server라는 하위 모듈이 private_ip라는 출력을 선언 한 경우 ㅐ당 값에 module.server.private_ip로 액세스할 수 있습니다.

### LOCAL VALUES

Local Value는 이름을 표현식에 지정하여 모듈 내에서 반복하지 않고 여러 번 사용할 수 있습니다.

#### Declaring a Local Value

Locals 블록을 사용하여 설정합니다.

<pre>
<code>
locals {
  service_name = "forum"
  owner        = "Community Team"
}
</code>
</pre>

간단한 상수이거나, 다른 모듈 또는 다른 리소스 값을 반환하거나, 복잡한 표현식도 가능합니다.

<pre>
<code>
locals {
  # 두개의 인스턴스의 ID List를 반환 합니다.
  instance_ids = concat(
    aws_instance.blue.*.id,
    aws_instance.green.*.id,
  )
}
locals {
  # 모든 자원에서 사용할 공통 태그
  common_tags = {
    Service = local.service_name
    Owner   = local.owner
  }
}

locals {
  # vpc 를 만들도록 설정 했으면 생성된 리소스에서 id 를 가져오고,
  # 만들지 않 도록 설정 헸으면 변수에서 가져 온다.
  vpc_id = var.create_vpc ? element(concat(aws_vpc.this.*.id, [""]), 0) : var.vpc_id
}
</code>
</pre>

### DATA SOURCES
  
Data Source를 통해 Terraform 구성의 다른곳에서 사용하기 위해 데이터를 가져오거나 계산할 수 있습니다. 데이터 소스를 사용하면 Terraform 구성이 Terraform 외부에서 정의되거나 다른 별도의 Terraform 구성으로 정의된 정보를 사용할 수 있습니다.

#### Using Data Sources

data 블록을 사용하여 설정합니다.

<pre>
<code>
data "aws_ami" "example" {
  owners = ["self"]
  tags = {
    Name   = "app-server"
    Tested = "true"
  }
  most_recent = true
}
</code>
</pre>

Terraform이 지정된 데이터 소스 aws_ami에서 읽고 해당 로컬 이름 example으로 결과를 내보내도록 요청합니다.
블록 본문 내에 ( { 와 } 사이 ) 데이터 소스에 의해 정의된 쿼리 제한 조건이 있습니다. 이 섹션의 대부분의 인수는 데이터 소스에 따라 다르며 실제로 이 예에서 most_recent, owners 및 tags는 모두 aws_ami 데이터 소스에 대해 별히 정의된 인수입니다.

<pre>
<code>
 data "aws_ami" "worker" {
  owners = ["602401143452"] # Amazon Account ID
  filter {
    name   = "name"
    values = ["amazon-eks-node-${var.kubernetes_version}-*"]
  }
  most_recent = true
}
</code>
</pre>

<pre>
<code>
data "aws_availability_zones" "azs" {
}
data "aws_caller_identity" "current" {
}
</code>
</pre>

### MODULES
  
module은 함께 사용되는 여러 리소스의 컨테이너입니다.

모든 Terraform 구성에는 기본 작업 디렉토리의 .tf 파일에 정의 된 리소스로 구성된 루트 모듈이라고 하는 하나 이상의 모듈이 있습니다.
모듈은 다른 모듈을 호출 할 수 있으며, 이른 통해 하위 모듈의 리소스를 간결하게 구성에 포함시킬 수 있습니다. 동일한 구성 내에서 또는 별도의 구성으로 모듈을 여러 번 호출하여 리소스 구성을 패키지화 하고 재사용할 수 있습니다.

#### Calling a Child Module

Module 블록을 사용하여 설정합니다.

<pre>
<code>
module "servers" {
  source = "./sub_module"
  servers = 5
}
</code>
</pre>

module 키워드는 바로 뒤에 있는 레이블은 로컬 이름이며 호출 모듈은 이 모듈 인스턴스를 참조하는데 사용할 수 있습니다.
모든 모듈에는 Terraform CLI에 의해 정의된 메타 인수인 source 인수가 필요합니다. 그 값은 모듈 구성 파일의 로컬 디렉토리에 대한 경로이거나 Terraform이 다운로드하여 사용해야 하는 원격 모듈 소스입니다.

<pre>
<code>
module "vpc" {
  source = "github.com/nalbam/terraform-aws-vpc?ref=v0.12.22"
  region = var.region
  name   = var.name
  vpc_id   = var.vpc_id
  vpc_cidr = var.vpc_cidr
}
</code>
</pre>

깃헙 주소 https://github.com/nalbam/terraform-aws-vpc 에서 v0.12.22 Tag를 지정하여 사용할 수 있습니다.

#### Accessing Module Output Values

모듈에 정의된 리소스는 캡슐화 되므로 호출 모듈은 해당 속성에 직접 액세스 할 수 없습니다. 그러나 자식 모듈은 출력값을 선언하여 호출 모듈이 액세스할 특정값을 선택적으로 내보낼 수 있습니다.

<pre>
<code>
resource "aws_elb" "example" {
  // ...
  instances = module.servers.instance_ids
}
</code>
</pre>

## COMMANDS (CLI)

Terraform은 명령 줄 인터페이스 (CLI)를 통해 제어됩니다.
Terraform은 단일 명령 행 애플리케이션 인 terraform 입니다. 그리고 apply 또는 plan과 같은 하위 명령을 사용합니다.
Terraform 명령을 실행하면 다음과 같은 설명이 출력 됩니다.

```  
<pre>
<code>
Usage: terraform [-version] [-help] <command> [args]
The available commands for execution are listed below.
The most common, useful commands are shown first, followed by
less common or more advanced commands. If you're just getting
started with Terraform, stick with the common commands. For the
other commands, please read the help and docs before usage.
Common commands:
    apply              Builds or changes infrastructure
    console            Interactive console for Terraform interpolations
    destroy            Destroy Terraform-managed infrastructure
    env                Workspace management
    fmt                Rewrites config files to canonical format
    get                Download and install modules for the configuration
    graph              Create a visual graph of Terraform resources
    import             Import existing infrastructure into Terraform
    init               Initialize a Terraform working directory
    output             Read an output from a state file
    plan               Generate and show an execution plan
    providers          Prints a tree of the providers used in the configuration
    refresh            Update local state file against real resources
    show               Inspect Terraform state or plan
    taint              Manually mark a resource for recreation
    untaint            Manually unmark a resource as tainted
    validate           Validates the Terraform files
    version            Prints the Terraform version
    workspace          Workspace management
All other commands:
    0.12upgrade        Rewrites pre-0.12 module source code for v0.12
    debug              Debug output management (experimental)
    force-unlock       Manually unlock the terraform state
    push               Obsolete command for Terraform Enterprise legacy (v1)
    state              Advanced state management
</code>
</pre>
```

* Environment Variables
* Apply
* Destroy
* Fmt
* Graph
* Import
* Init
* Output
* Plan
* Providers
* state
  
### ENVIRONMENT VARIABLES

Terraform은 동작의 다양한 측면을 사용자 정의하기 위한 여러 환경변수를 사용할 수 있습니다.

#### TF_VAR_name

환경변수를 사용하여 변수를 설정할 수 있습니다. 환경변수는 TF_VAR_name 형식입니다.

```hcl
export TF_VAR_region='us-west-1'
export TF_VAR_ami='ami-049d8641'
export TF_VAR_alist='[1, 2, 3]'
export TF_VAR_amap='{ foo = "bar", baz = "qux" }'
```

### APPLY
  
terraform apply 명령은 원하는 구성 상태에 도달하는데 필요한 변경사항 또는 terraform plan 실행 계획에 의해 생성된 사전 결정된 조치 세트를 적용하는데 사용됩니다.

#### Usage

terraform apply [options] [dir-or-plan]
기본적으로 apply는 구성에 대한 현재 디렉토리를 스캔하고 변경 사항을 적절하게 적용합니다.

#### Options

-input=true - 직접 설정되지 않은 경우 변수 입력을 요청합니다.
-auto-approve - 신청하기 전에 대화식 계획 승인을 건너 뜁니다.
-var 'foo=bar' - Terraform 구성에서 변수를 설정합니다. 이 플래그는 여러 번 설정할 수 있습니다. 변수 삾은 HCL로 해석되므로 이 플래그를 통해 목록 및 맵 값을 지정할 수 있습니다.
-var-file=foo - Terraform 구성의 변수를 변수 파일에서 설정합니다. Terraform.tfvars 또는 .auto.tfvars 파일이 현재 디렉토리에 있으면 자동으로 로드됩니다. 이 플래그는 여러 번 사용할 수 있습니다.
  
### DESTROY
  
terraform destroy 명령은 Terraform 관리 인프라를 삭제하는데 사용됩니다.

#### Usage

```
terraform destroy [options] [dir]
```

Terraform에서 관리하는 인프라가 파괴됩니다. 파괴하기 전에 확인을 요청합니다.
이 명령은 계획 파일 인수를 제외하고 apply 명령이 수락하는 모든 인수 및 플래그를 승인합니다.
-auto-approve 가 설정되면 삭제 확인이 표시되지 않습니다.
"종속성"에 영향을주는 대신 -target 플래그는 지정된 대상에 의존하는 모든 자원도 삭제합니다. 자세한 내용은 terraform plan의 target 문서를 참조하십시오.
terraform destroy 명령의 동작은 terraform plan -destroy 명령으로 언제든지 미리 볼 수 있습니다.

### FMT
  
terraform rmt 명령은 Terraform 구성 파일을 표준 형식 및 스타일로 다시 쓰는데 사용됩니다. 이 명령은 Terraform 언어 스타일 규칙의 하위 집합과 함께 가독성을 위한 기타 작은 조정 사항을 적용합니다.

#### Usage

```
terraform fmt [options] [dir]
```

Terraform 리소스의 시각적 종속성 그래프를 출력합니다.
그래포는 DOT 형식으로 출력됩니다. 이 형식을 읽을수 있는 일반적인 프로그램은 GraphViz이지만 이 형식을 읽을수 있는 웹 서비스도 있습니다.

#### Generating Images

```
terraform graph | dot -Tpng > graph.png
terraform graph | dot -Tsvg > graph.svg
```

### IMPORT
  
terraform import 명령은 기존 리소스를 Terraform으로 가져 오는데 사용됩니다.

#### Usage

```  
terraform import [options] ADDRESS ID
```

Import는 ID에서 기존 리소스를 찾아 주어진 ADDRESS에 Terraform 상태로 가져옵니다.
ADDRESS는 유효한 리소스 주소여야 합니다. 모든 자원 주소가 유효하기 때문에 import 명령은 자원을 모듈로 가져오거나 상태의 루트로 직접 가져올 수 있습니다.

ID는 가져오는 자원유형에 따라 다릅니다. 예를들어 AWS 인스턴스의 경우 인턴스 ID (i-abcd1234)이지만 AWS Route53 영역의 경우 영역 ID (Z12ABC4UGMOZ2N)입니다. ID 형식에 ㅐ한 자세한 내용은 제공 업체 설명서를 참조하십시오. 확실하지 않은 경우 ID를 사용해 보십시오. ID가 유효하지 않으면 오류 메시지만 표시됩니다.

#### Example: Import into Resource

이 예제는 AWS 인스턴스를 foo라는 aws_instance 리소스로 가져옵니다.

```
terraform import aws_instance.foo i-abcd1234
```

#### Example: Import into Module

아래 예제는 AWS 인스턴스를 bar라는 aws_instance 리소스로 foo라는 모듈로 가져옵니다.

```  
terraform import module.foo.aws_instance.bar i-abcd1234
```

#### Example: Import into Resource configured with count

아래 예제는 인스턴스를 count로 구성된 baz라는 aws_instance 리소스의 첫번째 인스턴스로 가져옵니다.

```  
terraform import 'aws_instance.baz[0]' i-abcd1234
```

### INIT
  
terraform init 명령은 Terraform 구성 파일이 포함된 작업 디렉토리를 초기화 하는데 사용됩니다. 새 Terraform 구성을 작성하거나 기존 버전 구성을 버전 제어에서 복제한 후에 실행해야하는 첫번째 명령입니다. 이 명령을 여러 번 실행하는것이 안전합니다.

#### Usage

```  
terraform init [options] [dir]
```

### OUTPUT

terraform output 명령은 상태 파일에서 출력 변수의 값을 추출하는데 사용됩니다.

#### Usage

```  
Terraform output [options] [NAME]
```

#### Options

-json - 지정된 경우 출력은 출력당 키를 사용하여 JSON 오브젝트로 형식화 됩니다. NAME을 정하면 지정된 출력만 반환됩니다. 추가 처리를 위해 jq와 같은 도구에 파이프로 연결할 수 있습니다.

### PLAN  
  
terraform plan 명령은 실행 계획을 작성하는데 사용됩니다. Terraform은 명시적으로 비활성화되지 않은 경우 새로 고침을 수행한 다음 구성 파일에 지정된 원하는 상태를 달성하는데 필요한 작업을 결정합니다.

이 명령은 실제 자원이나 상태를 변경하지 않고 일련의 변경에 대한 실행 계획이 예상과 일치하는지 확인하는 편리한 방법입니다.

#### Usage

```  
Terraform plan [option] [dir]
```

#### Options

-destroy - 설정된 경우 알려진 모든 자원을 삭제하는 계획을 생성합니다.
-input=true - 직접 설정되지 않은 경 변수 입력을 요청합니다.
-target=resource - 타게팅할 리소스 주소입니다. 이 플래그는 여러 번 사용할 수 있습니다. 자세한 내용은 아래를 참조하십시오.
-var 'foo=bar' - Terraform 구성에서 변수를 설정합니다.
-var-file=foo - Terraform 구성의 변수를 변수 파일에서 설정합니다. Terraform.tfvars 또는 .auto.tfvars 파일이 현재 디렉토리에 있으면 자동으로 로드됩니다. 이 플래그는 여러 번 사용할 수 있습니다.

#### Resource Targeting

-target 옵션을 사용하려면 리소스의 하위 집합에만 Terraform의 주의를 집중시킬 수 있습니다. 자원 주소 구문은 제한 조건을 지정하는데 사용됩니다.
이 타겟팅 기능은 실수 복구 또는 Terraform 제한 문제 해결과 같은 예외적인 상황을 위해 제공됩니다. 일상적인 작업에는 -target을 사용하지 않는것이 좋습니다. 이로 인해 구성 드리프트가 감지되지 않고 실제 자원 상태가 구성과 어떻게 관련 되는지 혼동될 수 있습니다.
  
### PROVIDERS  
 
terraform provider 명령은 현재 구성에 사용된 제공자에 대한 정보를 출력합니다.

#### Usage

```
terraform providers [config-path]
```

#### Example

```  
 . 
├── provider.aws
├── provider.terraform
└── module.eks
          ├── provider.aws (inherited)
          ├── provider.kubernetes
          ├── provider.local
          └── provider.template
```
  
```
 .
├── provider.aws
├── provider.terraform 
└── module.worker
         ├── provider.aws (inherited)
         └── module.worker
                  └── provider.aws (inherited)
```

### STATE

terraform state 명령은 고급 상태 관리에 사용됩니다. Terraform 사용이 향상됨에 따라 Terraform 상태를 수정해야하는 경우가 있습니다. 상태를 직접 수정하는 대신 terraform state 명령을 사용할 수 있습니다.

이 명령은 중첩된 하위 명령이며 추가 하위 명령이 있습니다.

#### Usage

```
terraform state <subcommand> [options] [args]
```

##### subcommands

* list
* mv
* pull
* push
* rm
* show

#### Remote State

Terraform state 부속 명령은 모두 마치 로컬 상태인 것처럼 원격 상태에서 작동합니다. 각 읽기 및 쓰기가 전체 네트워크 왕복을 수행하므로 읽기 및 쓰기가 평소보다 오래 걸릴 수 있습니다.

## PROVIDER
  
Terraform은 물리적 시스템, VM, 네트워크 스위치, 컨테이너 등과 같은 인프라 리소스를 생성, 관리 및 업데이트하는 데 사용됩니다. 거의 모든 인프라 유형이 Terraform의 리소스로 표현 될 수 있습니다.

### AWS PROVIDER
  
AWS (Amazon Web Services) 공급자는 AWS에서 지원하는 많은 리소스와 상호 작용하는데 사용됩니다. 공급자를 사용하려면 적절한 자격 증명으로 공급자를 구성해야 합니다.
아래 그림은 AWS Provider를 선언한 예시 코드입니다.

```hcl 
# Configure the AWS Provider
provider"aws"{
  region ="ap-northeast-2"}
# Create a VPC
resource"aws_vpc" "example"{
  cidr_block ="10.0.0.0/16"}
```

#### Authentication

AWS 공급자는 인증을 위한 신임 정보를 제공하는 유연한 수단을 제공합니다. 다음과 같은 방법이 순서대로 지원되며 아래에 설명되어 있습니다.

* Static credentials
* Environment variables
* Shared credentials file
* EC2 Role

##### Static credentials

※ info
모든 Terraform 구성에 대한 자격 증명 하드 코딩은 권장되지 않으며 이 파일이 공개 버전 제어 시스템에 커밋되면 비밀 유출 위험이 있습니다.

```hcl
provider"aws"{
  region     ="ap-northeast-2"access_key ="my-access-key"secret_key ="my-secret-key"}
```

##### Environment variable

Aws Access Key 및 AWS Secret Key를 각각 나타내는 AWS_ACCESS_KEY__ID 및 AWS_SECRET_ACCESS_KEY 환경 변수를 통해 자격 증명을 제공할 수 있습니다.

```hcl
provider"aws"{}
```

```hcl
export AWS_ACCESS_KEY_ID="my-access-key" export AWS_SECRET_ACCESS_KEY="my-secret-key" export AWS_DEFAULT_REGION="ap-northeast-2" terraform plan
```

##### Shared credentials file

AWS 자격 증명 파일을 사용하여 자격 명을 지정할 수 있습니다. 기본 위치는 Linux 및 OS X에서 $HOME/.aws/credentials 이거나 Windows 사용자의 경우 %USERPROFILE%\.aws\credentials 입니다.

```hcl
provider"aws"{
  region                  ="us-west-2"shared_credentials_file ="/Users/tf_user/.aws/cred-custom"profile                 ="custom-profile"}
```

##### EC2 Role

IAM 역할을 사용하여 IAM 인스턴스 프로파일이 있는 EC2 인스턴스에서 Terraform을 실행중인 경우 Terraform은 메타 데이터 API 엔드 ㅗ인트에 자격 증명을 요청하여 권한을 획득할 수 있습니다.

##### Assume role

역할 ARN이 제공되면 Terraform은 제공된 자격 증명을 사용하여 이 역할을 맡습니다.

```hcl  
provider"aws"{
  assume_role{
    role_arn     ="arn:aws:iam::ACCOUNT_ID:role/ROLE_NAME"session_name ="SESSION_NAME"external_id  ="EXTERNAL_ID"}
}
```

## PROVISIONERS
  
Provisioner를 사용하면 서버 또는 기타 인프라 개체를 서비스 할 수 있도록 로컬 컴퓨터 또는 원격 ㅓㅁ퓨터에서 특정 작업을 모델링 할 수 있습니다.

#### Provisioners are a Last Resort

Terraform에는 Terraform의 선언적 모델에서 직접 표현할 수 없는 특정 행동이 항상 있을것 임을 알고 프로비저닝의 개념을 실용주의의 척도로 포함합니다.
하지만 이것은 Terraform 사용에 상당한 복잡성과 불확실성을 추가합니다. 최대한 Terraform에서 제공되는 기본 기능으로 시도하고, 다른 옵션이 없을 경우에만 Provisioner를 사용하는 것이 좋습니다.
  
### PROVISIONER CONNECTION SETTINGS  

대부분의 프로 비져는 SSH 또는 WinRM을 통해 원격 리소스에 액세스 해야하며 연결 방법에 대한 세부 정보가 포함된 중첩된 연결 블록이 필요합니다.

#### Example

```hcl
# Copies the file as the root user using SSH
provisioner"file"{
  source      ="conf/myapp.conf"destination ="/etc/myapp.conf"connection{
    type     ="ssh"user     ="root"password ="${var.root_password}"host     ="${var.host}"}
}
```

### PROVISIONERS WITHOUT A RESOURCE
  
특정 리소스와 직접 연결되지 않은 프로 비저를 실행해야 하는 경우 해당 공급자를 null_resource와 연결할 수 있습니다.

#### Example

```hcl  
resource"aws_instance" "cluster"{
  count =3
  # ...
}
resource"null_resource" "cluster"{
  # Changes to any instance of the cluster requires re-provisioning
triggers ={
    cluster_instance_ids ="${join(",", aws_instance.cluster.*.id)}"}
  # Bootstrap script can run on any instance of the cluster
  # So we just choose the first in this case
connection{
    host ="${element(aws_instance.cluster.*.public_ip, 0)}"}
provisioner"remote-exec"{
    # Bootstrap script called with private_ip of each node in the cluster
inline =[
      "bootstrap-cluster.sh ${join(" ", aws_instance.cluster.*.private_ip)}",
    ]
  }
}
```

#### Example - kubernetes provider 사용 전

```hcl 
resource"null_resource" "executor"{
  depends_on =[aws_eks_cluster.cluster]
triggers ={
    endpoint =aws_eks_cluster.cluster.endpoint}
provisioner"local-exec"{
    working_dir =path.modulecommand =<<EOSecho"kubectl apply -f aws-auth.yaml"EOSinterpreter =var.local_exec_interpreter}
}
```

#### Example - kubernetes provider 사용 후

```hcl
resource"kubernetes_config_map" "aws_auth"{
  depends_on =[aws_eks_cluster.cluster]
metadata{
    name      ="aws-auth"namespace ="kube-system"}
data ={
    mapRoles =yamlencode(var.map_roles)
    mapUsers =yamlencode(var.map_users)
  }
}
```

## MODULES

모듈은 함께 사용되는 여러 리소스의 컨테이너입니다. 모듈을 사용하여 단한 추상화를 생성 할 수 있으므로 물리적 객체의 관점이 아닌 아키텍처의 관점에서 인프라를 설명 할 수 있습니다.
Terraform plan 또는 terraform apply시 작업 디렉토리의 .tf 파일은 루트 모듈에서 함께 적용됩니다. 해당 모듈은 다른 모듈을 ㅗ출하고 하나의 출력 값을 다른 모듈의 입력 값으로 전달하여 서로 연결할 수 있습니다.

```
$ tree sample-module/
 . 
├── README.md
├── main.tf 
├── variables.tf 
├── outputs.tf 
├── ... 
├── examples/ 
│      ├── main.tf 
│      ├── .../
```

* Module Sources
* Module Composition

### MODULE SOURCES
  
모듈 블록의 소스 인수는 Terraform에게 원하는 하위 모듈의 소스 코드를 찾을 수 있는 위치를 알려줍니다.
Terraform은 terraform init의 모듈 설치 단계에서 이를 사용하여 소스 코드를 로컬 디스크의 디렉토리에 다운로드하여 다른 Terraform 명령에서 사용할 수 있도록 합니다.
모듈 설치 프로그램은 아래 나열된 다양한 소스 유형에서 설치를 지원합니다.

#### Local paths

로컬 경로 참조를 사용하면 단일 소스 리포지토리 내 구성의 일부를 가져올 수 있습니다.

```hcl
module"consul"{
  source ="./consul"}
```

#### GitHub

Terraform은 접두사가 없는 github.com URL을 인식하여 자동으로 Git 리포지토리 소스로 해석합니다.

```hcl
module"consul"{
  source ="github.com/hashicorp/example"}
```

SSH를 통해 복제하려면 다음 형식을 사용하십시오.

```hcl
module"consul"{
  source ="git@github.com:hashicorp/example.git"}
```

#### Generic Git, Mercurial repositories

임의의 Git 리포지토리는 주소 앞에 특수한 git:: 접두사를 붙여서 사용할 수 있습니다. HTTPS 또는 SSH를 사용하려면 다음의 방법을 사용하십시오.
  
```hcl
module"vpc"{
  source ="git::https://example.com/vpc.git"}
module"storage"{
  source ="git::ssh://username@example.com/storage.git"}
```

#### S3 buckets

특수한 s3:: 접두어와 경로 스타일 S3 버킷 객체 URL을 사용하여 S3에 저장된 아카이브를 모듈 소스로 사용할 수 있습니다.

```hcl  
module"consul"{
  source ="s3::https://s3-eu-west-1.amazonaws.com/examplecorp-terraform-modules/vpc.zip"}
```
  
### MODULE COMPOSITION
  
모듈 블록의 소스 인수는 Terraform에게 원하는 하위 모듈의 소스 코드를 찾을 수 있는 위치를 알려줍니다.
Terraform은 terraform init의 모듈 설치 단계에서 이를 사용하여 소스 코드를 로컬 디스크의 디렉토리에 다운로드하여 다른 Terraform 명령에서 사용할 수 있도록 합니다.
모듈 설치 프로그램은 아래 나열된 다양한 소스 유형에서 설치를 지원합니다.

#### Local paths

로컬 경로 참조를 사용하면 단일 소스 리포지토리 내 구성의 일부를 가져올 수 있습니다.

```hcl
module"consul"{
  source ="./consul"}
```

#### GitHub

Terraform은 접두사가 없는 github.com URL을 인식하여 자동으로 Git 리포지토리 소스로 해석합니다.

```hcl
module"consul"{
  source ="github.com/hashicorp/example"}
```

SSH를 통해 복제하려면 다음 형식을 사용하십시오.

```hcl
module"consul"{
  source ="git@github.com:hashicorp/example.git"}
```

#### Generic Git, Mercurial repositories

임의의 Git 리포지토리는 주소 앞에 특수한 git:: 접두사를 붙여서 사용할 수 있습니다. HTTPS 또는 SSH를 사용하려면 다음의 방법을 사용하십시오.

```hcl
module"vpc"{
  source ="git::https://example.com/vpc.git"}
module"storage"{
  source ="git::ssh://username@example.com/storage.git"}
```

#### S3 buckets

특수한 s3:: 접두어와 경로 스타일 S3 버킷 객체 URL을 사용하여 S3에 저장된 아카이브를 모듈 소스로 사용할 수 있습니다.

```hcl
module"consul"{
  source ="s3::https://s3-eu-west-1.amazonaws.com/examplecorp-terraform-modules/vpc.zip"}
```
### MODULE COMPOSITION
  
모듈 블록의 소스 인수는 Terraform에게 원하는 하위 모듈의 소스 코드를 찾을 수 있는 위치를 알려줍니다.
Terraform은 terraform init의 모듈 설치 단계에서 이를 사용하여 소스 코드를 로컬 디스크의 디렉토리에 다운로드하여 다른 Terraform 명령에서 사용할 수 있도록 합니다.
모듈 설치 프로그램은 아래 나열된 다양한 소스 유형에서 설치를 지원합니다.

#### Local paths

로컬 경로 참조를 사용하면 단일 소스 리포지토리 내 구성의 일부를 가져올 수 있습니다.

```hcl  
module"consul"{
  source ="./consul"}
```

#### GitHub

Terraform은 접두사가 없는 github.com URL을 인식하여 자동으로 Git 리포지토리 소스로 해석합니다.

```hcl  
module"consul"{
  source ="github.com/hashicorp/example"}
```

SSH를 통해 복제하려면 다음 형식을 사용하십시오.

```hcl
module"consul"{
  source ="git@github.com:hashicorp/example.git"}
```

#### Generic Git, Mercurial repositories

임의의 Git 리포지토리는 주소 앞에 특수한 git:: 접두사를 붙여서 사용할 수 있습니다. HTTPS 또는 SSH를 사용하려면 다음의 방법을 사용하십시오.

```hcl
module"vpc"{
  source ="git::https://example.com/vpc.git"}
module"storage"{
  source ="git::ssh://username@example.com/storage.git"}
```

#### S3 buckets

특수한 s3:: 접두어와 경로 스타일 S3 버킷 객체 URL을 사용하여 S3에 저장된 아카이브를 모듈 소스로 사용할 수 있습니다.

```hcl
module"consul"{
  source ="s3::https://s3-eu-west-1.amazonaws.com/examplecorp-terraform-modules/vpc.zip"}
```
  
### STATE
  
Terraform은 인프라 및 구성에 대한 상태를 저장해야 합니다. 이 상태는 Terraform에서 실제 리소스를 구성에 매핑하고 메타 데이터를 추적하며 대규모 인프라의 성능을 향상 시키는데 사용 됩니다.
이 상태는 기본적으로 terraform.tfstate 라는 로컬 파일에 저장되지만 원격으로 저장할 수도 있으므로 팀 환경에서 더 잘 작동합니다.

* Purpose of Terraform State
* Remote State
* State Locking
  
### PURPOSE OF TERRAFORM STATE
  
Terraform이 작동하려면 상태가 필수 요건입니다. 이 페이지는 Terraform 상태가 필한 이유를 설명합니다.

#### Mapping to the Real World

Terraform은 Terraform 구성을 실제 세계에 매핑하기 위해 일종의 데이터베이스가 필요합니다. 여러분의 설정에 리소스 "aws_instance" "foo"가 있는 경우 Terraform은 이 맵을 사용하여 인스턴스 i-abcd1234가 해당 리소스로 표시되어 있음을 알 수 있습니다.

AWS와 같은 일부 공급자의 경우 Terraform은 이론적으로 AWS 태그와 같은 것을 사용할 수 있습니다. 하지만 모든 리소스가 태그를 지원하는 것은 아니며 모든 클라우드 제공 업체가 태그를 지원하는 것은 아닙니다.

따라서 실제 환경의 리소스에 구성을 매핑하기 위해 Terraform은 자체 상태 구조를 사용합니다.

#### Metadata

Terraform은 리소스와 원격 객체 간의 매핑과 함께 리소스 종속성과 같은 메타 데이터도 추적해야합니다.

Terraform은 일반적으로 구성을 사용하여 종속성 순서를 결정합니다. 그러나 Terraform 구성ㅓ 리소스를 삭제할 때 Terraform은 ㅐ당 리소스를 삭제하는 방법을 알아야합니다. Terraform은 구성에 없는 자원에 대한 맵핑이 존재하며 destroy plan을 볼 수 있습니다.

#### Performance

기본 매핑 외에 Terraform은 상태의 모든 리소스에 대한 속성 값의 캐시를 저장합니다.

#### Syncing

기본 구성에서 Terraform은 Terraform이 실행 된 현재 작업 디렉토리의 파일에 상태를 저장합니다. 팀에서 Terraform을 사용할 때는 모든 사람이 동일한 상태로 작업하여 동일한 원격 객체에 작업을 적용하는 것이 중요합니다.

### REMOTE STATE
  
기본적으로 Terraform은 terraform.tfstate 라는 파일에 상태를 로컬로 저장합니다. 팀에서 Terraform으로 작업할 때 로컬 파일을 사용하면 각 사용자가 Terraform을 실행하기 전에 항상 최신 상태 데이터를 가지고 있는지 확인하고 동시에 다른 사람이 Terraform을 실행하지 않도록 해야하므로 Terraform 사용이 복잡해집니다.

원격 상태에서 Terraform은 상태 데이터를 원격 데이f터 저장소에 기록한 다음 팀의 모든 구성원 간에 공유할 수 있습니다. Terraform은 Terraform Cloud, HashiCorp Consul, Amazon S3, Alibaba Cloud OSS 등의 상태 저장을 지원합니다.

#### Delegation and Teamwork

원격 상태는 더 쉬운 버전 제어 및 안전한 저장 이상을 제공합니다. 또한 출력을다른 팀에 위임 할 수도 있습니다. 이를 통해 인프라를 여러 팀이 액세스 할수 있는 구성 요소로 보다 쉽게 분류 할 수 있습니다.

다시 말해 원격 상태를 통해 팀은 추가 구성 장소에 의존하지 않고 인프라 리소스를 읽기 전용 방식으로 공유할 수 있습니다.

#### Locking and Teamwork

완벽한 기능을 갖춘 원격 백엔드의 경우 Terraform은 상태 잠금을 사용하여 동일한 상태에 대한 Terraform의 동시 실행을 방지할 수 있습니다.

#### Example

```hcl
terraform{
  backend"s3"{
    region ="ap-northeast-2"bucket ="terraform-mz-seoul"key    ="vpc-demo.tfstate"}
}
module"vpc"{
  source ="github.com/nalbam/terraform-aws-vpc?ref=v0.12.24"region =var.regionname   =var.name}
```
  
```hcl
data"terraform_remote_state" "vpc"{
  backend ="s3"config ={
    region ="ap-northeast-2"bucket ="terraform-mz-seoul"key    ="vpc-demo.tfstate"}
}
module"eks"{
  source ="github.com/nalbam/terraform-aws-eks?ref=v0.12.32"region =var.regionname   =var.namevpc_id     =data.terraform_remote_state.vpc.outputs.vpc_idsubnet_ids =data.terraform_remote_state.vpc.outputs.subnet_ids}
```
 
### STATE LOCKING
  
백엔드에서 지원하는 경우 Terraform은 상태를 기록할 수 있는 모든 작업에 대해 상태를 잠급니다. 이렇게하면 다른 사람이 잠금을 획득하여 잠재적으로 상태를 손상시킬 수 없습니다.
상태 잠금은 상태를 기록할 수 있는 모든 작업에서 자동을 발생합니다. -lock 플래그를 사용하여 대부분의 명령에 대해 상태 잠금을 비활성화 할 수 있지만 권장되지는 않습니다.![image](https://user-images.githubusercontent.com/66822357/186339587-bc51a0a3-e8b4-4c74-9d33-6e36c8ebd3d0.png)

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

* cout: 개수에 따라 여러 자원을 작성
* depend_on: 숨겨진 종속성을 명시적으로 지정
* for_each: 맵 또는 문자열 세트에 따라 여러 자원을 성
* provider: 기본이 아닌 공급자 구성을 선탱
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

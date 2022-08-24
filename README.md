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

## PROVIDERS
  
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


  
  
  
  
  
  ## RESOURCE

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

'''
resource"aws_vpc" "main"{
  cidr_block =var.base_cidr_block}
BLOCK_TYPE"BLOCK_LABEL" "BLOCK_LABEL"{
  // Block body
IDENTIFIER =EXPRESSION # Argument
}
'''

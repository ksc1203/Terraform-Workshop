백엔드는 상태를 저장하고 상태 잠금을 위한 API를 제공합니다. 상태 잠금은 선택 사항입니다.
아래는 Amazon S3를 원격 상태 저장소로 사용하고, DynamoDB를 잠금 기능으로 선언한 예시 코드입니다.

```hcl
terraform{
  backend"s3"{
    region         ="ap-northeast-2"bucket         ="terraform-mz-seoul"key            ="vpc-demo.tfstate"dynamodb_table ="terraform-mz-seoul"encrypt        =true}
}
```

### [첫 페이지](https://github.com/EstebanHan/Terraform-Workshop)

### [Docs 페이지](https://github.com/EstebanHan/Terraform-Workshop/tree/main/DOCS/01_Configuration_Language)

### [Backends](https://github.com/EstebanHan/Terraform-Workshop/tree/main/DOCS/07_Backends)
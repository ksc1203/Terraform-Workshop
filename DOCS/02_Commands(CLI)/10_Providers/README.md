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

### [첫 페이지](https://github.com/EstebanHan/Terraform-Workshop)

### [Docs 페이지](https://github.com/EstebanHan/Terraform-Workshop/tree/main/DOCS)

### [Commands(CLI)](https://github.com/EstebanHan/Terraform-Workshop/tree/main/DOCS/02_Commands(CLI))
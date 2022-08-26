대부분의 프로 비져는 SSH 또는 WinRM을 통해 원격 리소스에 액세스 해야하며 연결 방법에 대한 세부 정보가 포함된 중첩된 연결 블록이 필요합니다.

#### Example

```hcl
# Copies the file as the root user using SSH
provisioner"file"{
  source      ="conf/myapp.conf"destination ="/etc/myapp.conf"connection{
    type     ="ssh"user     ="root"password ="${var.root_password}"host     ="${var.host}"}
}
```
### [첫 페이지](https://github.com/EstebanHan/Terraform-Workshop)

### [Docs 페이지](https://github.com/EstebanHan/Terraform-Workshop/tree/main/DOCS/01_Configuration_Language)

### [Provisioners](https://github.com/EstebanHan/Terraform-Workshop/tree/main/DOCS/04_Provisioners)
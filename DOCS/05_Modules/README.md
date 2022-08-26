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

* [Module Sources](https://github.com/EstebanHan/Terraform-Workshop/tree/main/DOCS/05_Modules/01_Module_Sources)
* [Module Composition](https://github.com/EstebanHan/Terraform-Workshop/tree/main/DOCS/05_Modules/02_Module_Composition)

### [첫 페이지](https://github.com/EstebanHan/Terraform-Workshop)

### [Docs 페이지](https://github.com/EstebanHan/Terraform-Workshop/tree/main/DOCS/01_Configuration_Language)

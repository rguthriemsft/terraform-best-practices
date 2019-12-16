
## Manage Blob Storage backend for tfstate files

Terraform doesn't support [Interpolated variables in terraform backend config](https://github.com/hashicorp/terraform/pull/12067), normally you write a seperate script to define a backend storage name for different environments, but I recommend to hard code it.

Add below code in terraform configuration files.
```
$ cat main.tf

terraform {
  required_version = "~> 0.12"

  backend "azurerm" {
    encrypt = true
  }
}
```

Define backend variables for particular environment
```
$ cat config/backend-dev.conf
storage_account_name  = "<unique_storage_account_name>-terraform-development"
container_name = "tfstate"
key     = "development/service-1.tfstate"
encrypt = true
region  = "westus2"
access_key="<access_key>"
```

### Notes
- storage_account_name - Azure Storage account name, has to be globally unique.
- container_name - he name of the blob container.
- key - Set some meaningful names for different services and applications, such as vpc.tfstate, application_name.tfstate, etc
- access_key - Storage Account Access Key


After you set `config/backend-dev.conf` and `config/dev.tfvars` properly (for each environment). You can easily run terraform as below:

```
env=dev
terraform get -update=true
terraform init -backend-config=config/backend-${env}.conf
terraform plan -var-file=config/${env}.tfvars
terraform apply -var-file=config/${env}.tfvars
```

#### Resources
* [Azure RM Backend Documentation](https://www.terraform.io/docs/backends/types/azurerm.html)
* [Tutorial: Store Terraform state in Azure Storage](https://docs.microsoft.com/en-us/azure/terraform/terraform-backend)
* [State Locking](https://www.terraform.io/docs/state/locking.html)

## Retrieve state meta data from a remote backend

Normally we have several layers to manage terraform resources, such as network, database, application layers. After you create the basic network resources, such as vnet, network security group and subnets, your database layer and applications layer should always refer the resource from the vnet layer directly via `terraform_remote_state` data srouce. 

>Notes: in Terraform v0.12+, you need add extra `outputs` to reference the attributes, otherwise you will get error message of [Unsupported attribute](https://github.com/hashicorp/terraform/issues/21442)

```
data "terraform_remote_state" "vpc" {
  backend = "s3"
  config = {
    bucket = var.s3_terraform_bucket
    key    = "${var.environment}/vpc.tfstate"
    region = var.aws_region
  }
}
 
# Retrieves the vpc_id and subnet_ids directly from remote backend state files.
resource "aws_xx_xxxx" "main" {
  # ...
  subnet_ids = split(",", data.terraform_remote_state.vpc.data_subnets)
  vpc_id     = data.terraform_remote_state.vpc.outputs.vpc_id
}
```
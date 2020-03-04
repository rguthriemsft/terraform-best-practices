# Terraform best practices on Azure

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Terraform best practices on Azure](#terraform-best-practices-on-azure)
  - [Run terraform command with var-file](#run-terraform-command-with-var-file)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Run terraform command with var-file

``` BASH
cat config/dev.tfvars

name = "dev"
resource_group_name = "global-terraform-state-rg"
storage_account_name = "dev-terraform-state"
tag_environment = "dev"
tag_version = "3.0.0"

terraform plan -var-file=config/dev.tfvars
```

With `var-file`, you can easily manage environment (dev/stag/uat/prod) variables.

With `var-file`, you avoid running terraform with long list of key-value pairs ( `-var foo=bar` )

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

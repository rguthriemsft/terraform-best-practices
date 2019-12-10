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

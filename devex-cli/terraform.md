---
description: DevEx sub-command for Terraform
---

# Terraform

The Terraform tools are offered by the `devex terraform` \(or `devex tf` for short\) command.
This command offers a set of common options that apply to all sub-commands.
The available sub-commands are:

* `taint-secrets` \(`ts` for short\): Taint all secrets in a terraform module state.

## Taint Secrets

Terraform is able to create resources that are secrets or credentials that should be rotated regularly.
The rotation is particularly efficient if those secrets are also propagated by terraform to the paces they need to be used at.
For example, with terraform we can create AWS api key pairs and populate those as GitHub Actions secrets or GitLab Pipeline variables.
Rotating those secrets is a matter of tainting the AWS api key pair resource in the Terraform state and then running `terraform apply`.

What this tool offers is an automated way of [tainting all resources in a Terraform state](https://www.terraform.io/docs/cli/state/taint.html) that are deemed to have created secrets.
The tool does not execute the rotation itself as it does not run `terraform apply`, but it leaves the resources tainted in the state so that in the next `terraform apply` over the respective module the secrets will be recreated.

The command takes one parameter, the local path to the terraform module.
The path parameter is required.

```text
devex terraform taint-secrets [-huvVx] [--debug] [--trace] MODULE
```

Besides the common `devex` command line options, the `terraform taint-secrets` command offers the following options.

* `-u`, `--untaint`: Untaint previously tainted secrets. This effectively reverses the actions of the tool.

### Tainting terraform secrets

To taint secrets in the state of a terraform module the module needs to be reachable in the local filesystem, and it also needs to be filly initialized.
Any credentials required to access the state (ie. to execute `terraform state pull|push`) need to be provided in the exact same way they are provided to `terraform state` commands.

Assuming there is a module at `~/terraform/infrastructure` we can taint the secrets on that module with the following command.
```text
devex terraform taint-secrets ~/terraform/infrastructure
```

If to execute a `terraform state` command we would need to provide some credentials via the environment, then we need to provide the same credentials to `devex terraform taint-secrets`.
We will use an aws named profile as an example of passing credentials via the environment.
```text
# terraform state command with aws named profile
AWS_PROFILE=my-aws-profile terraform state pull

# devex terraform taint-secrets with the same aws named profile
AWS_PROFILE=my-aws-profile devex terraform taint-secrets ~/terraform/infrastructure
```

### Untainting secrets

To revert the changes made by the tool it is possible to set the `--untaint` flag while re-running the tool.
In that case the tool will untain all secrets, effectively resetting the state back.
```text
devex terraform taint-secrets --untaint ~/terraform/infrastructure
```
However, this is only possible if no other operation manipulated the state in between taint and untaint.

### What are secrets?

Secrets are terraform resources that contain sensitive material and because of that it's good to re-generate them periodically.
There are many terraform providers and each defines resources that can be seen as secrets.
The tool has a default list of what resources are secrets that should keep evolving.
The default list is defined in a [configuration file](https://github.com/miguelaferreira/devex-cli/blob/main/src/main/resources/application.yml).
Feel free to open pull-requests to add new secrets to the default list.
However, it is possible to overwrite the list at runtime by setting a comma separated list of resource types in the variable `TERRAFORM_SECRETS`.

In the following command we overwrite the list of secrets the tool will consider as secrets.
```text
TERRAFORM_SECRETS="aws_iam_access_key,tls_private_key,random_password" devex terraform taint-secrets MODULE
```

### Terragrunt and other terraform wrappers

The tool expect that the terraform binary is installed and discoverable via the environment's `PATH` variable.
However, when it is not directly discoverable it is possible to define the full path to the binary using the `TERRAFORM_COMMAND` variable.
```text
TERRAFORM_COMMAND="/Users/miguel/tools/terraform" devex terraform taint-secrets MODULE
```

Using the same mechanism it is possible to use terraform wrappers like terragrunt instead.
If terragrunt is discoverable via the environment's `PATH` then just setting the terraform command to `terragrunt` will be enough.
```text
TERRAFORM_COMMAND="terragrunt" devex terraform taint-secrets MODULE
```
Otherwise, use the full path to the `terragrunt` binary.

### Rotating secrets

Tainting the resources does not actually re-create them.
All it does is mark them for recreation on the next `terraform apply`.
To effectively rotate the secrets follow the taint-secrets command with a `terraform apply`.

```text
devex terraform taint-secrets ~/terraform/infrastructure
cd ~/terraform/infrastructure
terraform apply
```

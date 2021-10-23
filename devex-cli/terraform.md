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

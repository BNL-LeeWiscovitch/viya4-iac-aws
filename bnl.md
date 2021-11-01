starting over from scratch in order to use official repos to setup the environment and cluster...then we'll analyze and scale down.

# tools

## aws cli

already previously installed and configured

config is set for us-east-2 and credentials are (overly) sufficient

## kubectl

already previously installed and configured

## terraform

```shell
sudo yum install -y yum-utils && \
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo && \
sudo yum -y install terraform
```

verify it's functional

```shell
terraform --version
...
Terraform v1.0.10
on linux_amd64
```

### aws credentials

we'll pass the pre-configured aws information to terraform using these environment variables:

```shell
export TF_VAR_aws_profile=default && \
export TF_VAR_aws_shared_credentials_file=~/.aws/credentials
```

## jq

not sure what this is yet, but it's listed as a terraform requirement. official repos only have 1.5, so need to manually install latest (1.6):

```shell
wget https://github.com/stedolan/jq/releases/download/jq-1.6/jq-linux64 && \
sudo mv jq-linux64 /usr/bin/jq && \
sudo chmod +x /usr/bin/jq
```

verify it's functional

```shell
jq --version
...
jq-1.6
```

# aws environment

we'll be using this [repo](https://github.com/sassoftware/viya4-iac-aws) along with terraform to create the default resources.

clone the repo:

```shell
cd ~ && \
git clone git@github.com:BNL-LeeWiscovitch/viya4-iac-aws.git && \
cd viya4-iac-aws
```

create terraform.tfvars and use the contents of examples\sample-input-minimum.tfvars to start. the variables in this file will overright the default ones provided by variables.tf. specific changes needed to be made:

```yaml
prefix                                  = "viya"
location                                = "us-east-2"
ssh_public_key                          = "~/.ssh/bnl-cloud-20210302.pem"
default_public_access_cidrs             = ["10.20.0.0/16"]
tags                                    = { "customer" = "bnl", "project" = "viya", "owner" = "sysadmin", "environment" = "development" }

postgres_servers = {
  default = {
    administrator_password       = "<bitwarden: bnl-eks-viya-psql>"
  },
}
```

>since terraform.tfvars is on the ignore list, need to manually create it on the admin-pl-cli02 host

## initialize the plan

```shell
terraform init
```

```txt
Initializing modules...
- autoscaling in modules/aws_autoscaling
Downloading terraform-aws-modules/iam/aws 4.1.0 for autoscaling.iam_assumable_role_with_oidc...
- autoscaling.iam_assumable_role_with_oidc in .terraform/modules/autoscaling.iam_assumable_role_with_oidc/modules/iam-assumable-role-with-oidc
Downloading terraform-aws-modules/eks/aws 17.1.0 for eks...
- eks in .terraform/modules/eks
- eks.fargate in .terraform/modules/eks/modules/fargate
- eks.node_groups in .terraform/modules/eks/modules/node_groups
- jump in modules/aws_vm
- kubeconfig in modules/kubeconfig
- nfs in modules/aws_vm
Downloading terraform-aws-modules/rds/aws 3.3.0 for postgresql...
- postgresql in .terraform/modules/postgresql
- postgresql.db_instance in .terraform/modules/postgresql/modules/db_instance
- postgresql.db_option_group in .terraform/modules/postgresql/modules/db_option_group
- postgresql.db_parameter_group in .terraform/modules/postgresql/modules/db_parameter_group
- postgresql.db_subnet_group in .terraform/modules/postgresql/modules/db_subnet_group
- vpc in modules/aws_vpc

Initializing the backend...

Initializing provider plugins...
- Finding terraform-aws-modules/http versions matching ">= 2.4.1"...
- Finding hashicorp/null versions matching "3.1.0"...
- Finding hashicorp/random versions matching ">= 2.2.0, >= 3.1.0, 3.1.0"...
- Finding latest version of hashicorp/cloudinit...
- Finding hashicorp/template versions matching "2.2.0"...
- Finding hashicorp/aws versions matching ">= 2.23.0, >= 2.49.0, >= 3.40.0, >= 3.43.0, 3.43.0"...
- Finding hashicorp/local versions matching ">= 1.4.0, 2.1.0"...
- Finding hashicorp/external versions matching "2.1.0"...
- Finding hashicorp/kubernetes versions matching ">= 1.11.1, 2.2.0"...
- Installing terraform-aws-modules/http v2.4.1...
- Installed terraform-aws-modules/http v2.4.1 (self-signed, key ID B2C1C0641B6B0EB7)
- Installing hashicorp/null v3.1.0...
- Installed hashicorp/null v3.1.0 (signed by HashiCorp)
- Installing hashicorp/template v2.2.0...
- Installed hashicorp/template v2.2.0 (signed by HashiCorp)
- Installing hashicorp/aws v3.43.0...
- Installed hashicorp/aws v3.43.0 (signed by HashiCorp)
- Installing hashicorp/external v2.1.0...
- Installed hashicorp/external v2.1.0 (signed by HashiCorp)
- Installing hashicorp/random v3.1.0...
- Installed hashicorp/random v3.1.0 (signed by HashiCorp)
- Installing hashicorp/cloudinit v2.2.0...
- Installed hashicorp/cloudinit v2.2.0 (signed by HashiCorp)
- Installing hashicorp/local v2.1.0...
- Installed hashicorp/local v2.1.0 (signed by HashiCorp)
- Installing hashicorp/kubernetes v2.2.0...
- Installed hashicorp/kubernetes v2.2.0 (signed by HashiCorp)

Partner and community providers are signed by their developers.
If you'd like to know more about provider signing, you can read about it here:
https://www.terraform.io/docs/cli/plugins/signing.html

Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

## review what is going to be made

```shell
terraform plan -out=terraform.output
```
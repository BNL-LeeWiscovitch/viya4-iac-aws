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
prefix                                  = "viya_"
location                                = "us-east-2"
ssh_public_key                          = "~/.ssh/bnl-cloud-20210302.pem"
default_public_access_cidrs             = ["10.20.0.0/16"]
tags                                    = { "customer" = "bnl", "project" = "viya", "owner" = "sysadmin", "environment" = "development" }

postgres_servers = {
  default = {
    administrator_password       = "P@ssw0rd123"
  },
}
```
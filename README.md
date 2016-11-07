# Amazon Web Services VPC Terraform Module

This Terraform module creates a configurable general purpose [Amazon Web Services VPC](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_Introduction.html). The module offers an opinionated but flexible network topography geared towards general purpose situations with separate public and private subnets. Each VPC can be configured to support one to four availability zones. Private subnet [NAT](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/vpc-nat.html) can be configured via either [NAT Gateways](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/vpc-nat-gateway.html). A single [Internet Gateway](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_Internet_Gateway.html) is created to provide public routing for public subnets. The module does not configure a bastion or VPN instance for private subnet instance access.

This module has been tested with Terraform version 0.7.1.

## Example VPC Layout: 3 AZ's

![Example VPC: 3AZ](https://dl.dropboxusercontent.com/s/0z9vrjrjsb51cq5/example-vpc-3AZ.png)

## Usage

* Include the module in your `main.tf`:

```
module "vpc" {
  source = "https://github.com/reactiveops/terraform-vpc.git"

  aws_access_key = "${var.aws_access_key}"
  aws_secret_key = "${var.aws_secret_key}"
  aws_region = "${var.aws_region}"

  az_count =  "${var.az_count}"
  aws_azs = "${var.aws_azs}"

  vpc_cidr_base = "${var.vpc_cidr_base}"

}
```

* Create the required variables either in `main.tf` or a separate `variables.tf` file:

```
variable "aws_access_key" {}
variable "aws_secret_key" {}
variable "aws_region" {}

variable "aws_azs" {}
variable "az_count" {}

variable "vpc_cidr_base" {}

```

* Assign variable values, for example in a `terraform.tfvars` file:

```
aws_azs = "us-west-2a, us-west-2b, us-west-2c, us-west-2d"
az_count = 3
vpc_cidr_base = "10.0"
```

This repo contains a few example `*.tfvars.examples` files showing example variable configurations.

## Configuration Options

### VPC IP Addresses

Generated VPC's will have a /16 CIDR block providing up to 65,536 IP addresses. Choose the IP range you want by setting the `vpc_cidr_base` variable to the first two numbers of the desired IP range. All subnets will use IP CIDR's built on this pattern.

```
vpc_cidr_base = "10.1"
```

The following subnets will be created in each AZ:

* Public
  * Resources requiring public IP addresses like VPN or bastion instances and Elastic Load Balancers.
* Private working
  * Internal non-production resources like web server and database instances.
* Private production
  * Internal production resources like web server and database instances.
* Private admin
  * Internal shared administrative resources like build server instances.

Each subnet will be a /21 block providing up to 2,048 IP addresses per subnet and AZ.

### AZ Count

Your VPC can span between one and four AZ's. You can select the specific AZ's that should be used.

```
aws_azs = "us-west-2a, us-west-2b, us-west-2c, us-west-2d"
az_count = 4
```

## Testing

This repo contains a few `.tfvars.example` files in the root illustrating different module usage configuration patterns. Each `.tfvars.example` file has a corresponding tfplan output file under `test/fixtures` representing the expected output. The project Makefile includes targets for installing a specific version of Terraform and comparing results of a `terraform plan` against expected output files.

### Setup

Running `make test` requires an actual AWS account for plan generation. The AWS account used requires read-only access to VPC/EC2 resources. No changes are applied. Credentials can be set via environment variables.

```
export TF_VAR_aws_access_key=XXXXXXXXXXXXXXXXX
export TF_VAR_aws_secret_key=XXXXXXXXXXXXXXXXXXXXXXXXXXXX
```

### Executing tests

```
> make test
```

Makefile defaults expect execution on OS X. To execute on Linux:

```
> make test TF_PLATFORM=Linux
```

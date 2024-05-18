# Terraform AWS Deployment Guide

Welcome to the Terraform AWS Deployment Guide. This guide walks you through the process of setting up and deploying infrastructure on AWS using Terraform. We cover everything from pre-requisites, configuring Terraform with AWS, to deploying a VPC and an EC2 instance. Let's get started.

## A. Pre-Requisites

Before you begin, ensure you have the following ready:

- An AWS account along with your access key and secret.
- AWS CLI configured on your machine.
- Terraform CLI installed and configured.
- Access to your S3 bucket for state storage.

## B. Configure Terraform with Default Region to Frankfurt

1. **Set up AWS CLI Credentials**: Run `aws configure` and set `eu-central-1` as the default region, providing your access key and secret when prompted.

2. **Create `import.tf` File**: Inside a new folder named `s3-bucket`, create an `import.tf` file with the following content:

    ```hcl
    resource "aws_s3_bucket" "my-state-bucket" {
      bucket = "bkt-ligaac-w3-euc1-ligaac${var.user-name}"

      tags = {
        Owner       = var.user-name
        Environment = "LigaAC-labs"
        Account     = "Workload3"
      }
    }
    ```

3. **Create `variables.tf` File**:

    ```hcl
    variable "user-name" {
        type = string
    }
    ```

4. **Create `providers.tf` File**:

    ```hcl
    provider "aws" {
      region = "eu-central-1"
    }
    ```

5. **Create `terraform.tfvars.json` File**: Populate this file with your user shortname.

    ```json
    {
        "user-name": "savua"
    }
    ```

6. **Import S3 Bucket in Terraform**:

    - Initialize Terraform: `terraform init`
    - Plan your deployment (showing the auto tfvars part): `terraform plan --var-file=terraform.tfvars.json`
    - Import the S3 bucket: `terraform import aws_s3_bucket.my-state-bucket bkt-ligaac-w3-euc1-savua`
    - Apply changes (visualize enforcement of tags): `terraform apply --var-file=terraform.tfvars.json`

## C. Deploy a VPC

1. **Prepare Configuration**: Copy `providers.tf`, `variables.tf`, and `terraform.tfvars.json` files into the new root folder.

2. **Create `backend.tf` File**: This file points to your S3 bucket for state storage(change with your username).

    ```hcl
    terraform {
      backend "s3" {
        bucket = "bkt-ligaac-w3-euc1-ligaacsavua"
        key    = "terraform/state"
        region = "eu-central-1"
      }
    }
    ```

3. **Create and Configure `main.tf` File**:

    - Add a data block for available zones.
    - Define locals for VPC name and tags.
    - Add the VPC module block as shown in the initial message.

4. **Run Terraform Commands**:

    - Initialize: `terraform init --upgrade`
    - Plan: `terraform plan --var-file=terraform.tfvars.json`
    - Apply: `terraform apply --var-file=terraform.tfvars.json`

## D. Deploy an EC2 Instance

1. **Update Locals**: Add `ec2_name` inside locals.

    ```hcl
    ec2_name = "ec2-${var.user-name}"
    ```

2. **Add EC2 Instance Module**: Reference the VPC module for `subnet_id`.

    ```hcl
    module "ec2_instance" {
      source  = "terraform-aws-modules/ec2-instance/aws"
      version = "5.6.1"

      name                        = local.ec2_name
      instance_type               = "t2.micro"
      subnet_id                   = module.vpc.private_subnets[0]
      associate_public_ip_address = false

      tags = local.tags
    }
    ```

3. **Run Terraform Commands**:

    - Initialize: `terraform init --upgrade`
    - Plan: `terraform plan --var-file=terraform.tfvars.json`
    - Apply: `terraform apply --var-file=terraform.tfvars.json`

## E. Destroy All Resources

1. **Run Terraform Commands**:

    - Destroy the resources: `terraform destroy`

2. **Review Execution**:

    - Check resource availability inside the AWS Console.
    - Review the state file contents in your S3 bucket.

## F. Format and Document

- Format your Terraform code: `terraform fmt --recursive`
- Generate documentation using `terraform-docs` as described in the initial message.

By following these steps, you'll have a solid foundation for managing AWS resources with Terraform. Always refer to the official Terraform and AWS documentation for more detailed information and best practices.


<!-- BEGIN_TF_DOCS -->
## Requirements

No requirements.

## Providers

| Name | Version |
|------|---------|
| <a name="provider_aws"></a> [aws](#provider\_aws) | 5.50.0 |

## Modules

| Name | Source | Version |
|------|--------|---------|
| <a name="module_ec2_instance"></a> [ec2\_instance](#module\_ec2\_instance) | terraform-aws-modules/ec2-instance/aws | 5.6.1 |
| <a name="module_vpc"></a> [vpc](#module\_vpc) | terraform-aws-modules/vpc/aws | 5.8.1 |

## Resources

| Name | Type |
|------|------|
| [aws_availability_zones.available](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/availability_zones) | data source |

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| <a name="input_user-name"></a> [user-name](#input\_user-name) | n/a | `string` | n/a | yes |

## Outputs

No outputs.
<!-- END_TF_DOCS -->
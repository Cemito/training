# Terraform

Terraform is an open-source Infrastructure as Code (IaC) tool It allows users to define and provision infrastructure
resources in a declarative configuration language called HashiCorp Configuration Language (HCL).

## Install Terraform

1. Install Terraform https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli

Verify Installation:

```
terraform --version
Terraform v1.7.5
```

## Create An IAM Role

In a new file called main.tf copy and paste:

```terraform
data "aws_iam_policy_document" "AccountRoleAssumption" {
  statement {
    actions = ["sts:AssumeRole"]
    principals {
      type        = "AWS"
      identifiers = [
        "arn:aws:iam::${var.account_id}:role/OrganizationAccountAccessRole"
      ]
    }
    effect = "Allow"
  }
}

# AccountUser

resource "aws_iam_role" "AccountUserRole" {
  name                 = "AccountUser"
  description          = "This role is managed by Tableau Public Cloud. You do not have permissions to modify this role."
  max_session_duration = 36000
  assume_role_policy   = data.aws_iam_policy_document.AccountRoleAssumption.json
}

### User-Policy

data "aws_iam_policy_document" "AccountUser-IAM-Policy" {
  statement {
    actions = [
      "s3:*",
      "dynamodb:*",
      "iam:ListUsers",
    ]
    resources = ["*"]
  }
}

resource "aws_iam_policy" "AccountUser-IAM-Policy" {
  name        = "UserPolicy"
  policy      = data.aws_iam_policy_document.AccountUser-IAM-Policy.json
  description = "Policy to allow AccountUser access via IDP"
  lifecycle {
    ignore_changes = [
      description
    ]
  }
}

resource "aws_iam_role_policy_attachment" "AccountUser-IAM-Policy" {
  role       = aws_iam_role.AccountUserRole.name
  policy_arn = aws_iam_policy.AccountUser-IAM-Policy.arn
}
```

In a new file called providers.tf

```terraform
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.0"
    }
  }
}

# Configure the AWS Provider
provider "aws" {
  region = "us-west-2"
}
```

## Terraform Plan

This command will show you a plan of what Terraform will create without actually doing it.

```
terraform plan
```

## Terraform Apply

This command will actually create the resources.

```
terraform apply
```
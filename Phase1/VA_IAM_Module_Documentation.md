---
title: "VA-IAM Module – VirtualAgentics"
status: "Draft"
audience: "Internal – DevOps/Infra Team, Contributors"
authors: "VirtualAgentics Infrastructure Team"
version: "0.1"
date: "2025-06-3"
gpt_model: "Drafted by ChatGPT."
---

# VA-IAM Terraform Module

This document describes the VA-IAM Terraform module, which provisions AWS Identity and Access Management (IAM) roles and policies for the VirtualAgentics Phase 1 infrastructure, consolidating identity management and ensuring secure operational practices.

## Overview

The **VA-IAM** module provides centralized management of IAM roles and policies essential for Phase 1 of VirtualAgentics infrastructure. It facilitates secure and efficient operation of the CI/CD pipeline, Lambda functions, and administrative tasks by enforcing strict role-based access controls and least-privilege security principles, enabling clear separation of duties and streamlined identity governance.

## Scope/Purpose

This module creates and configures IAM resources including:

- **Terraform CI/CD Role:** Enables secure Terraform deployment via GitHub Actions using OIDC trust, eliminating long-lived credentials.
- **Lambda Execution Roles:** Defined for agent Lambdas (e.g., ContentGen, Review, Publish), enabling logging, accessing Secrets Manager, S3 storage, and service-specific permissions.
- **Administrative Roles:** Includes security audit or read-only roles as per baseline AWS Control Tower requirements or additional Phase 1 needs.

Role naming follows the VirtualAgentics standard (`va-prod-<purpose>-role`), ensuring consistency and clarity.

## Assumptions/Constraints

- Designed specifically for the `prod` AWS account in Phase 1.
- Requires AWS Control Tower or existing account setup.
- Assumes existing or separate creation of GitHub OIDC provider in IAM if not handled directly by this module.
- Secrets (e.g., API keys) managed exclusively through AWS Secrets Manager.
- No IAM user credentials are provisioned; only IAM roles and policies.

## Architecture

IAM roles integrate directly with key Phase 1 components:

- **Terraform CI/CD role:** GitHub Actions assumes this role via OIDC to perform Terraform apply operations. Policy attachments typically include fine-grained or managed policies such as `AdministratorAccess` (temporary or highly scoped policies recommended).
- **Lambda Execution roles:** Provide access to AWS services necessary for specific agent Lambdas, structured using a least-privilege model.

Example relationship:
```
GitHub Actions OIDC → IAM Terraform CI/CD Role → AWS Permissions
Lambda functions → Lambda Execution Role → AWS Permissions
```

## Usage Example

```hcl
module "iam" {
  source                      = "./modules/va-iam"
  environment                 = "prod"
  terraform_oidc_provider_url = "token.actions.githubusercontent.com"
  terraform_ci_role_name      = "va-prod-core-terraform-iam-role"
  terraform_ci_role_policy_arns = ["arn:aws:iam::aws:policy/AdministratorAccess"]

  lambda_roles = {
    contentgen = {
      name     = "va-prod-contentgen-lambda-role"
      policies = ["arn:aws:iam::aws:policy/AWSLambdaBasicExecutionRole", "arn:aws:iam::123456789012:policy/va-prod-contentgen-policy"]
    }
  }

  tags = {
    Project     = "VirtualAgentics"
    Environment = "prod"
    Owner       = "infra"
    Purpose     = "phase1-iam"
  }
}
```

## Inputs

| Name                           | Type           | Default                              | Description                                                   |
|--------------------------------|----------------|--------------------------------------|---------------------------------------------------------------|
| `environment`                  | string         | -                                    | Environment name for naming resources (e.g., prod, dev).      |
| `create_oidc_provider`         | bool           | false                                | Creates GitHub OIDC provider if set to true.                  |
| `terraform_oidc_provider_url`  | string         | "token.actions.githubusercontent.com" | OIDC provider URL for GitHub Actions.                         |
| `terraform_ci_role_name`       | string         | "va-prod-core-terraform-iam-role"    | IAM role name for Terraform deployments.                      |
| `terraform_ci_role_policy_arns`| list(string)   | -                                    | ARNs of policies attached to Terraform CI role.               |
| `lambda_roles`                 | map(object)    | {}                                   | Definitions for Lambda roles including names and policies.    |
| `tags`                         | map(string)    | {}                                   | Tags to be applied to all IAM resources.                      |

## Outputs

| Output Name                 | Description                           |
|-----------------------------|---------------------------------------|
| `terraform_ci_role_arn`     | ARN of the Terraform CI/CD IAM role.  |
| `terraform_ci_role_name`    | Name of the Terraform CI/CD IAM role. |
| `lambda_roles_arns`         | Map of Lambda role names to ARNs.     |
| `oidc_provider_arn`         | ARN of created OIDC provider, if any. |

## Examples/Use Cases

- The ARN of the Terraform CI/CD role is configured in GitHub Actions workflows, facilitating secure infrastructure deployments.
- Lambda module references the IAM Lambda roles for secure function execution. Centralizing IAM roles simplifies management and auditing.

## Security Considerations

- IAM policies implement the principle of least privilege.
- Example: ContentGen Lambda policy permits Secrets Manager read (`GetSecretValue`), S3 write, and CloudWatch Logs access.
- Terraform CI role may require broad permissions initially, which should be audited rigorously.
- All IAM activities are logged via AWS CloudTrail.

## Testing

- Validate role assumption using AWS IAM Access Analyzer.
- Conduct dry-run role assumption from GitHub Actions pipeline.
- Verify Lambda permissions through test invocation (accessing Secrets Manager, writing to S3).
- Execute `terraform validate` and simulate policies with IAM simulators.

## Versioning

This module is maintained in the `virtualagentics-iac` repository and uses semantic versioning. Due to sensitivity, changes undergo careful review. Major version increments accompany breaking changes, including modifications to role names or policies.

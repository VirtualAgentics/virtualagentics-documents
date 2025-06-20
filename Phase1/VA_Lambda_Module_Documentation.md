---
title: "VA-Lambda Module – VirtualAgentics"
status: "Draft"
audience: "Internal – DevOps/Infra Team, Contributors"
authors: "VirtualAgentics Infrastructure Team"
version: "0.1"
date: "2025-06-3"
gpt_model: "Drafted by ChatGPT."
---

# VA-Lambda Terraform Module

This document details the VA-Lambda Terraform module, facilitating deployment of AWS Lambda functions for VirtualAgentics Phase 1, ensuring consistency, best practices, and simplified operations.

## Overview

The **VA-Lambda** module automates the deployment of serverless functions (Lambda) for VirtualAgentics Phase 1 agents including Content Generation, Review, and Publishing. It encapsulates common configurations such as runtime setup, IAM roles, and logging, enabling streamlined deployments with minimal duplication and maximum security.

## Scope/Purpose

This module provisions:

- **AWS Lambda Function:** Specified with runtime, handler, memory allocation, timeout, and code artifact location.
- **IAM Role and Policies:** Optionally creates a role with basic execution permissions and additional specified policies. External roles can be used by providing an ARN.
- **CloudWatch Log Group:** Explicitly managed with defined retention settings.
- **Event Triggers:** Optional creation of event triggers like schedules via EventBridge or service events (S3, DynamoDB).

## Assumptions/Constraints

- Lambda artifacts (zip files) must be available beforehand (via CI pipeline or S3).
- Infrastructure prerequisites (IAM roles, VPC, security groups) must exist if using external references.
- Secrets managed via AWS Secrets Manager; module does not handle creation, only references existing ARNs.

## Architecture Details

Each Lambda deployed via this module integrates into the agent-based system of VirtualAgentics. For example, the ContentGen Lambda could trigger from manual invocation, scheduled tasks, or API calls and perform actions such as interacting with DynamoDB, Secrets Manager, or S3.

## Usage Example

```hcl
module "contentgen_lambda" {
  source                 = "./modules/va-lambda"
  function_name          = "va-prod-contentgen-lambda"
  code_s3_bucket         = "va-prod-lambda-artifacts"
  code_s3_key            = "contentgen/v1.0/contentgen.zip"
  runtime                = "python3.9"
  handler                = "contentgen.handler"
  memory_size            = 512
  timeout                = 30
  environment_variables  = {
    "ENV" = "prod",
    "OPENAI_API_KEY_SECRET_ARN" = "arn:aws:secretsmanager:region:account-id:secret:va-prod-openai-api"
  }
  create_role            = true
  role_name              = "va-prod-contentgen-lambda-role"
  additional_policies    = ["arn:aws:iam::aws:policy/AWSLambdaBasicExecutionRole"]
  tags                   = {
    Project     = "VirtualAgentics",
    Environment = "prod",
    Owner       = "content-team",
    Purpose     = "content-generation-agent"
  }
}
```

## Inputs

| Name                    | Type         | Default | Description                                           |
|-------------------------|--------------|---------|-------------------------------------------------------|
| `function_name`         | string       | -       | Lambda function name (required).                      |
| `code_s3_bucket`        | string       | -       | Bucket where Lambda code artifact is stored.          |
| `code_s3_key`           | string       | -       | Path within bucket to Lambda artifact (required).     |
| `runtime`               | string       | -       | Runtime environment (e.g., python3.9).                |
| `handler`               | string       | -       | Function entry point (required).                      |
| `memory_size`           | number       | 128     | Memory size allocated to Lambda function.             |
| `timeout`               | number       | 30      | Lambda function execution timeout (seconds).          |
| `environment_variables` | map(string)  | {}      | Environment variables for Lambda execution context.   |
| `create_role`           | bool         | true    | Whether module creates IAM role.                      |
| `role_name`             | string       | -       | Name of IAM role if created.                          |
| `additional_policies`   | list(string) | []      | IAM policies attached to created role.                |
| `existing_role_arn`     | string       | -       | ARN of external IAM role if not created by module.    |
| `tags`                  | map(string)  | {}      | Tags applied to all resources.                        |

## Outputs

| Output Name            | Description                     |
|------------------------|---------------------------------|
| `lambda_function_arn`  | ARN of deployed Lambda function.|
| `lambda_function_name` | Name of Lambda function.        |
| `role_arn`             | ARN of IAM role if created.     |

## Example Scenarios

- Deploying Review Lambda with similar patterns.
- Scheduled Lambda (Publish agent) invoked daily using EventBridge rule (supported by additional module parameters).

## Dependencies/Prerequisites

- S3 artifact bucket must exist.
- Required IAM roles/VPC resources must be pre-created.
- Any additional resources (S3, DynamoDB) referenced in Lambda code must exist prior.

## Testing

- Validate deployment in AWS Lambda console.
- Trigger manually or via configured events.
- Verify IAM permissions (Secrets Manager, S3).
- Check CloudWatch Logs for proper logging.
- Execute `terraform validate` to ensure correctness.

## Versioning

Semantic versioning in `virtualagentics-iac` repository. Lambda function code versioning handled separately by CI/CD pipelines; module focuses on infrastructure changes and enhancements.

---
title: "VA-S3 Module – VirtualAgentics"
status: "Stable Draft"
audience: "Internal – DevOps/Infra Team, Contributors"
authors: "VirtualAgentics Infrastructure Team"
version: "1.0"
date: "2025-06-3"
gpt_model: "Drafted by ChatGPT."
---

# VA-S3 Terraform Module

This document outlines the VA-S3 Terraform module, responsible for provisioning Amazon S3 buckets and DynamoDB tables to support VirtualAgentics Phase 1, ensuring secure and standardized storage infrastructure.

## Overview

The **VA-S3** module creates critical storage infrastructure including the Terraform state bucket, application data buckets, and the DynamoDB table for Terraform state locking. It ensures secure, reliable, and consistent setup according to best practices, crucial for maintaining state management and data storage integrity.

## Scope/Purpose

The module provisions:

- **Terraform State S3 Bucket:** Versioned, encrypted storage for Terraform state files named conventionally (`va-prod-terraform-state`). Secure bucket policy and blocked public access.

- **Terraform State Lock DynamoDB Table:** Used for concurrency control during Terraform operations (`va-prod-terraform-lock`). Simple key schema (`LockID`).

- **Application/Data S3 Buckets:** Example bucket (`va-prod-content-bucket`) for content generation outputs, private, encrypted, with optional lifecycle policies.

## Assumptions/Constraints

- Requires pre-configured AWS environment and IAM roles.
- Bucket names must be globally unique.
- Initial Terraform state bucket setup uses local state before switching to remote backend.
- Versioning enabled, with implications for managing historical data.

## Usage Example

```hcl
module "s3" {
  source                         = "./modules/va-s3"
  environment                    = "prod"
  create_state_bucket            = true
  state_bucket_name              = "va-prod-terraform-state"
  enable_versioning              = true
  enable_default_encryption      = true
  create_lock_table              = true
  lock_table_name                = "va-prod-terraform-lock"
  create_content_bucket          = true
  content_bucket_name            = "va-prod-content-bucket"
  content_bucket_versioning      = true
  content_bucket_lifecycle_rules = [
    { id = "cleanup-old-content", prefix = "", enabled = true, expiration_days = 90 }
  ]
  tags = {
    Project     = "VirtualAgentics",
    Environment = "prod",
    Owner       = "infra",
    Purpose     = "state-and-storage"
  }
}
```

## Inputs

| Name                            | Type           | Default | Description                                    |
|---------------------------------|----------------|---------|------------------------------------------------|
| `environment`                   | string         | -       | Environment for resource naming.               |
| `create_state_bucket`           | bool           | true    | Creates the Terraform state bucket.            |
| `state_bucket_name`             | string         | -       | Name of the state bucket.                      |
| `enable_versioning`             | bool           | true    | Enables bucket versioning.                     |
| `enable_default_encryption`     | bool           | true    | Enables default encryption (AES-256).          |
| `create_lock_table`             | bool           | true    | Creates DynamoDB lock table.                   |
| `lock_table_name`               | string         | -       | Name of the lock table.                        |
| `create_content_bucket`         | bool           | false   | Creates additional content bucket.             |
| `content_bucket_name`           | string         | -       | Name of the content bucket.                    |
| `content_bucket_versioning`     | bool           | true    | Enables content bucket versioning.             |
| `content_bucket_lifecycle_rules`| list(object)   | []      | Lifecycle policies for the content bucket.     |
| `tags`                          | map(string)    | {}      | Tags applied to resources.                     |

## Outputs

| Output Name         | Description                                        |
|---------------------|----------------------------------------------------|
| `state_bucket_id`   | ID of Terraform state bucket.                      |
| `lock_table_name`   | DynamoDB state lock table name.                    |
| `content_bucket_id` | ID of the content bucket created.                  |

## Examples/Use Cases

- Terraform backend configuration references state bucket and lock table.
- Content generation Lambdas store outputs to designated buckets.

## Security Considerations

- Encryption enabled and public access blocked for buckets.
- Terraform state protected by encryption and restrictive bucket policies.
- DynamoDB lock table ensures secure concurrency management.

## Testing

- Confirm bucket creation, encryption, and versioning via AWS console.
- Validate state locking functionality.
- Verify tags and lifecycle policies.

## Versioning

Managed using semantic versioning. All changes undergo careful review due to sensitive data handling.

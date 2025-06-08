
# AWS Control Tower Setup

## Prerequisites
- AWS root account.
- Administrator access.
- Verified domain and email addresses.

## Organizational Structure Diagram
The following diagram outlines the Virtual Agentics AWS account hierarchy and organizational units:

```mermaid
graph TD;
    ROOT["Virtual Agentics (Root)"]
    CORE["Core OU"] --> LOG_ARCHIVE["va-core-log-archive"]
    CORE --> AUDIT["va-core-audit"]
    CORE --> SECURITY["va-core-security"]

    PROD["Production OU"] --> PROD_APPS["va-prod-applications"]
    PROD --> PROD_AGENTS["va-prod-agents"]
    PROD --> PROD_DATA["va-prod-datastore"]

    DEV["Development OU"] --> DEV_SANDBOX["va-dev-sandbox"]
    DEV --> DEV_TEST["va-dev-testing"]

    SHARED["Shared Services OU"] --> SHARED_INFRA["va-shared-infrastructure"]
    SHARED --> SHARED_NET["va-shared-networking"]
    SHARED --> SHARED_OPS["va-shared-operations"]

    ROOT --> CORE
    ROOT --> PROD
    ROOT --> DEV
    ROOT --> SHARED
```

## Terraform Integration for Guardrails

Guardrails are enforced explicitly via Terraform. Below are examples for enabling common preventive and detective controls:

### Preventive Guardrail Example: Disallow Public S3 Buckets
```hcl
resource "awscc_controltower_control" "disallow_public_s3" {
  control_identifier = "AWS-GR_RESTRICTED_PUBLIC_BUCKETS"
  target_identifier  = aws_organizations_organizational_unit.prod_ou.id
}
```

### Detective Guardrail Example: Detect Root Account Usage
```hcl
resource "awscc_controltower_control" "detect_root_usage" {
  control_identifier = "AWS-GR_DETECT_ROOT_USER_LOGIN"
  target_identifier  = aws_organizations_organizational_unit.core_ou.id
}
```

### Terraform Account Creation & Baseline Setup
```hcl
resource "aws_organizations_account" "prod_agents" {
  name      = "va-prod-agents"
  email     = "va-prod-agents@virtualagentics.ai"
  parent_id = aws_organizations_organizational_unit.prod_ou.id
}

resource "aws_iam_role" "baseline_admin_role" {
  name = "VAProdBaselineAdminRole"

  assume_role_policy = jsonencode({
    Version = "2012-10-17",
    Statement = [{
      Effect = "Allow",
      Principal = {
        AWS = "arn:aws:iam::<Control Tower Management Account ID>:root"
      },
      Action = "sts:AssumeRole"
    }]
  })

  inline_policy {
    name = "BaselineAdminPermissions"
    policy = jsonencode({
      Version = "2012-10-17",
      Statement = [{
        Effect = "Allow",
        Action = ["*"],
        Resource = ["*"]
      }]
    })
  }
}
```

## Next Steps
- Validate Terraform code in a sandbox environment.
- Confirm permissions and guardrail effectiveness.
- Document any manual steps explicitly.

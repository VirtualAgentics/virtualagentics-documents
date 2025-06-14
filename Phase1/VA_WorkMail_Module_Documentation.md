---
title: "VA-WorkMail Module – VirtualAgentics"
status: "Stable Draft"
audience: "Internal – DevOps/Infra Team, Contributors"
authors: "VirtualAgentics Infrastructure Team"
version: "1.0"
date: "2025-06-3"
gpt_model: "Drafted by ChatGPT."
---

# VA-WorkMail Terraform Module

This documentation describes the VA-WorkMail Terraform module, which automates the setup of AWS WorkMail for VirtualAgentics, providing corporate email infrastructure.

## Overview

The **VA-WorkMail** module automates creation and configuration of the AWS WorkMail organization, domain associations, and user mailboxes. It ensures consistent deployment and verification of the custom email domain (virtualagentics.ai), critical for official communication and AWS account alias management in Phase 1.

## Scope/Purpose

The module provisions:

- **AWS WorkMail Organization:** Container for email resources, created in region eu-central-1, incurring minimal AWS costs.
- **WorkMail Domain Association:** Adds the domain `virtualagentics.ai` to the organization. Domain verification via DNS (handled externally through VA-Route53) is required.
- **WorkMail User/Mailbox:** Creates mailbox users such as `prod-mailbox` with display name "Prod Team Mailbox".
- **Email Alias Management:** Configures email aliases like `prod@virtualagentics.ai` forwarding to the created mailbox user, supporting AWS account email management.
- **Integration with SES:** Enables email sending via AWS SES, leveraging necessary DNS records (DKIM, SPF) managed by the VA-Route53 module.

## Assumptions/Constraints

- Assumes existing Route53 hosted zone for domain DNS verification.
- AWS WorkMail organization limits: typically one per region per account.
- Mailbox passwords should be managed securely, recommended via AWS Secrets Manager or Terraform sensitive variables.
- Domain verification status depends on DNS setup completion; verify in WorkMail console after Terraform apply.

## Usage Example

```hcl
module "workmail" {
  source                  = "./modules/va-workmail"
  organization_name       = "VirtualAgenticsProd"
  domain_name             = "virtualagentics.ai"
  create_mailbox_user     = true
  mailbox_user_name       = "prod-mailbox"
  mailbox_display_name    = "Prod Team Mailbox"
  mailbox_password        = var.workmail_mailbox_password
  enable_alias_forwarding = true
  alias_forward_target    = "prod-mailbox"
  tags = {
    Project     = "VirtualAgentics",
    Environment = "prod",
    Owner       = "infra",
    Purpose     = "workmail-setup"
  }
}
```

## Inputs

| Name                     | Type    | Default | Description                                  |
|--------------------------|---------|---------|----------------------------------------------|
| `organization_name`      | string  | -       | Name for WorkMail organization.              |
| `domain_name`            | string  | -       | Domain name associated with WorkMail.        |
| `create_mailbox_user`    | bool    | false   | Toggles creation of a mailbox user.          |
| `mailbox_user_name`      | string  | -       | Username for mailbox user.                   |
| `mailbox_display_name`   | string  | -       | Display name for mailbox user.               |
| `mailbox_password`       | string  | -       | Password for mailbox (securely handled).     |
| `enable_alias_forwarding`| bool    | false   | Enables alias forwarding to a mailbox user.  |
| `alias_forward_target`   | string  | -       | Mailbox user receiving alias forwarded emails.|
| `tags`                   | map     | {}      | Tags for module resources (if applicable).   |

## Outputs

| Output Name          | Description                            |
|----------------------|----------------------------------------|
| `organization_id`    | ID of created WorkMail organization.   |
| `mailbox_user_id`    | ID of the created mailbox user.        |
| `mailbox_email`      | Email address of created mailbox user. |

## Example Use Cases

- Establishing managed email (e.g., prod@virtualagentics.ai) as AWS root account contact.
- Future agent-driven communications via configured WorkMail mailbox.
- Ensuring centralized email management and notification capturing for operational purposes.

## Dependencies

- Requires DNS records setup via VA-Route53 module for domain verification.
- WorkMail availability within AWS account (Control Tower compliance checks recommended).
- Secure handling and provisioning of mailbox passwords via AWS Secrets Manager.

## Testing

- Verify WorkMail organization status in AWS console or CLI.
- Confirm domain verification through WorkMail domain list.
- Test inbound email reception by sending external test emails.
- Validate outbound email via WorkMail web client or email client setup.
- Confirm correct email alias and mailbox configurations.

## Security Considerations

- Secure password handling via Secrets Manager or Terraform variables.
- Domain verification and secure DNS configurations via VA-Route53 module.
- Audit WorkMail mailbox accesses and email logs periodically.

## Versioning

Module maintained using semantic versioning. Future mailbox or domain modifications will increment module version. Carefully review changes affecting email operations and domain configurations.

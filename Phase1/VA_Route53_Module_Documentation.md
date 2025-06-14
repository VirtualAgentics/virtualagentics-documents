---
title: "VA-Route53 Module – VirtualAgentics"
status: "Stable Draft"
audience: "Internal – DevOps/Infra Team, Contributors"
authors: "VirtualAgentics Infrastructure Team"
version: "1.0"
date: "2025-06-3"
gpt_model: "Drafted by ChatGPT."
---

# VA-Route53 Terraform Module

This document outlines the VA-Route53 Terraform module responsible for provisioning DNS configurations using AWS Route 53 for the VirtualAgentics project's foundational domain management.

## Overview

The **VA-Route53** module creates and manages the primary AWS Route 53 hosted zone for the organization's domain (*virtualagentics.ai*). Utilizing Infrastructure-as-Code ensures domain configurations remain version-controlled, reproducible, and reliably consistent throughout the lifecycle of the infrastructure.

## Scope/Purpose

The module provisions:

- **Hosted Zone Creation:** Establishes a public hosted zone for *virtualagentics.ai*. Requires updating NS records at the domain registrar if the domain isn't originally managed by Route 53.

- **DNS Records:** 
  - **MX record:** Configured for email delivery via AWS WorkMail/SES (e.g., *10 inbound-smtp.eu-central-1.amazonaws.com*).
  - **TXT records:** Domain verification and SES authentication.
  - **DKIM CNAME records:** Required for SES email authentication, typically consisting of three CNAME entries.
  - Optional records like SPF (TXT), Autodiscover (CNAME), and aliases for email management.

Future records like subdomains or web endpoints can easily be integrated in subsequent phases.

## Assumptions/Constraints

- Domain ownership (*virtualagentics.ai*) must be already established externally.
- NS records at external registrar must manually point to Route 53 NS servers created by this module.
- WorkMail setup is either already in place or executed concurrently to retrieve domain verification tokens. DNS records required by WorkMail (verification TXT, MX, DKIM) are provided as inputs to this module.

## Usage Example

```hcl
module "dns" {
  source        = "./modules/va-route53"
  domain_name   = "virtualagentics.ai"
  create_zone   = true
  zone_id       = null
  records       = [
    { name = "@", type = "MX", value = "10 inbound-smtp.eu-central-1.amazonaws.com", ttl = 300 },
    { name = "_amazonses", type = "TXT", value = ""random-token123"", ttl = 300 },
    { name = "randomstring1._domainkey", type = "CNAME", value = "randomstring1.dkim.amazonses.com", ttl = 300 }
  ]
  tags = {
    Project     = "VirtualAgentics",
    Environment = "prod",
    Owner       = "infra",
    Purpose     = "primary-dns"
  }
}
```

The `domain_name` is the root domain. The `records` list demonstrates adding MX, TXT, and DKIM CNAME entries required for WorkMail/SES setup.

## Inputs

| Name          | Type           | Default | Description                                            |
|---------------|----------------|---------|--------------------------------------------------------|
| `domain_name` | string         | -       | Primary domain name (e.g., virtualagentics.ai).        |
| `create_zone` | bool           | true    | Indicates if the hosted zone should be created.        |
| `zone_id`     | string         | null    | Existing hosted zone ID if `create_zone` is false.     |
| `records`     | list(map)      | []      | DNS records definitions (name, type, value, ttl).      |
| `tags`        | map(string)    | {}      | Tags applied to all resources.                         |

## Outputs

| Output Name   | Description                                      |
|---------------|--------------------------------------------------|
| `zone_id`     | Hosted zone ID created or managed by this module.|
| `zone_name`   | Domain name of the hosted zone.                  |
| `ns_records`  | Name servers for the hosted zone (to update at registrar). |

## Examples/Use Cases

Primarily used in Phase 1 for setting up the organization's main DNS zone, ensuring email functionality through WorkMail and SES. Additional subdomains and records for services introduced in subsequent phases can leverage this module similarly.

## Dependencies

- Requires DNS verification details from the VA-WorkMail module.
- Domain ownership at a registrar with ability to update NS records.
- SES integration is implicitly required (through WorkMail setup).

## Testing

- Validate the hosted zone and DNS records via AWS Route 53 console.
- Use DNS query tools (`nslookup`, `dig`) to confirm record propagation.
- Confirm domain verification and email functionality through WorkMail console and email tests.

## Versioning

Version-controlled within the `virtualagentics-iac` repository using semantic versioning. DNS changes should undergo careful review and validation to avoid downtime or configuration errors.

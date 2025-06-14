---
title: "VA-Security Module – VirtualAgentics"
status: "Stable Draft"
audience: "Internal – DevOps/Infra Team, Contributors"
authors: "VirtualAgentics Infrastructure Team"
version: "1.0"
date: "2025-06-3"
gpt_model: "Drafted by ChatGPT."
---

# VA-Security Terraform Module

This document covers the VA-Security Terraform module responsible for provisioning AWS Security Groups, establishing the network security baseline for the VirtualAgentics Phase 1 infrastructure.

## Overview

The **VA-Security** module defines AWS Security Groups (SGs) to enforce inbound and outbound traffic rules within the core VPC. Even though Phase 1 lacks internet-facing resources, establishing these security groups now prepares the environment and ensures internal components follow strict security guidelines.

## Scope/Purpose

The module provisions:

- **Lambda Security Group:** Allows no inbound connections; allows all outbound traffic, supporting external API calls from Lambda.
- **Placeholder Security Groups:** Pre-defined but currently inactive:
  - **ALB Security Group:** Intended for future Application Load Balancer with placeholder rules.
  - **Bastion Security Group:** Intended for future bastion host access via SSH from specific IP ranges.

This module does not alter the default VPC Security Group or Network ACLs (NACLs), using AWS default NACL rules in Phase 1.

## Assumptions/Constraints

- Requires existing VPC from the VA-VPC module.
- Security groups created are placeholders with minimal rules to adhere to the principle of least privilege. Some groups are currently not attached to resources.
- Parameterization for allowed IPs (e.g., bastion allowed CIDR) defaults to restricted or empty, ensuring no unnecessary access.

## Usage Example

```hcl
module "security" {
  source = "./modules/va-security"
  vpc_id = module.core_vpc.vpc_id
  create_lambda_sg   = true
  lambda_sg_name     = "va-prod-core-lambda-sg"
  allow_lambda_egress_cidr = "0.0.0.0/0"

  create_alb_sg      = true
  alb_sg_name        = "va-prod-alb-sg"
  alb_allow_cidrs    = ["0.0.0.0/0"]
  alb_allow_ports    = [443]

  create_bastion_sg  = true
  bastion_sg_name    = "va-prod-bastion-sg"
  bastion_allow_cidrs = ["203.0.113.0/24"]
  bastion_allow_ports = [22]

  tags = {
    Project     = "VirtualAgentics",
    Environment = "prod",
    Owner       = "security",
    Purpose     = "baseline-sg"
  }
}
```

## Inputs

| Name                      | Type         | Default     | Description                                              |
|---------------------------|--------------|-------------|----------------------------------------------------------|
| `vpc_id`                  | string       | -           | ID of the VPC where SGs are created (required).          |
| `create_lambda_sg`        | bool         | true        | Whether to create the Lambda SG.                         |
| `lambda_sg_name`          | string       | -           | Name for Lambda security group.                          |
| `allow_lambda_egress_cidr`| string       | "0.0.0.0/0" | Allowed outbound CIDR for Lambda.                        |
| `create_alb_sg`           | bool         | true        | Whether to create ALB SG.                                |
| `alb_sg_name`             | string       | -           | Name for ALB security group.                             |
| `alb_allow_cidrs`         | list(string) | []          | Allowed CIDRs for ALB inbound traffic (placeholder).     |
| `alb_allow_ports`         | list(number) | []          | Allowed ports for ALB (typically HTTPS).                 |
| `create_bastion_sg`       | bool         | true        | Whether to create Bastion SG.                            |
| `bastion_sg_name`         | string       | -           | Name for Bastion security group.                         |
| `bastion_allow_cidrs`     | list(string) | []          | Allowed CIDRs for Bastion SSH access (placeholder).      |
| `bastion_allow_ports`     | list(number) | []          | Allowed ports for Bastion (typically SSH, port 22).      |
| `tags`                    | map(string)  | {}          | Tags applied to all resources.                           |

## Outputs

| Output Name      | Description                        |
|------------------|------------------------------------|
| `lambda_sg_id`   | ID of created Lambda security group|
| `alb_sg_id`      | ID of created ALB security group   |
| `bastion_sg_id`  | ID of created Bastion security group|

## Example Use Cases

- VA-Lambda module uses `lambda_sg_id` for secure outbound Lambda traffic.
- Future ALB setup references `alb_sg_id` for incoming HTTPS traffic.
- Bastion SG available for manual or future automated EC2 troubleshooting instances.

## Security Considerations

- Default to no open ports, strict least privilege principle.
- Security groups limit inbound access severely; egress explicitly defined.
- Phase 1's placeholder SGs reduce future attack surfaces.

## Testing

- Verify SG existence and rules via AWS VPC console.
- Test Lambda outbound access if attached; future ALB and Bastion instances testable upon implementation.
- Confirm security settings via AWS CLI and Terraform outputs.

## Versioning

Semantic versioning in the `virtualagentics-iac` repository. Updates, especially when activating placeholders in future phases, are carefully versioned and reviewed.

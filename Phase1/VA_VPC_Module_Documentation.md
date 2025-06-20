---
title: "VA-VPC Terraform Module Documentation"
status: "Stable Draft"
audience: "Internal – DevOps/Infra Team, Contributors"
authors: "VirtualAgentics Infrastructure Team"
version: "1.0"
date: "2025-06-3"
gpt_model: "Drafted by ChatGPT."
---

# VA-VPC Terraform Module

This document describes the VA-VPC Terraform module, which provisions the foundational AWS Virtual Private Cloud (VPC) and subnet infrastructure in the VirtualAgentics production environment.

## Overview

The **VA-VPC** Terraform module establishes a secure, isolated, and structured networking environment within AWS, specifically tailored for VirtualAgentics' production workloads. The resulting VPC acts as the core network backbone for all agent-based workloads.

## Scope/Purpose

This module provisions:

- A VPC with a predefined CIDR block (e.g., `10.0.0.0/16` for prod core).
- Three-tier subnet architecture: Public, Private (with NAT Gateway), and Isolated subnets.
- Internet Gateway, NAT Gateways, and VPC Endpoints for AWS S3 and DynamoDB.

Subnet CIDRs follow VirtualAgentics' addressing plan.

## Assumptions/Constraints

- Intended for prod environment core VPC in Phase 1.
- Inputs required: `environment`, `vpc_cidr`, subnet CIDRs.
- Assumes at least three AWS AZs available.
- Explicitly manages AWS default resources.

## Architecture/Diagram

VPC structured with:

- Three public, private, and isolated subnets across AZs.
- Internet Gateway for public subnet access.
- NAT Gateways for private subnet egress.
- VPC Endpoints for AWS services.

## Usage Example

```hcl
module "core_vpc" {
  source                = "./modules/va-vpc"
  environment           = "prod"
  vpc_cidr              = "10.0.0.0/16"
  public_subnet_cidrs   = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  private_subnet_cidrs  = ["10.0.11.0/24", "10.0.12.0/24", "10.0.13.0/24"]
  isolated_subnet_cidrs = ["10.0.21.0/24", "10.0.22.0/24", "10.0.23.0/24"]
  enable_nat_gateway    = true
  create_endpoints      = true
  tags = {
    Project     = "VirtualAgentics"
    Environment = "prod"
    Owner       = "infra"
    Purpose     = "core-app-network"
  }
}
```

## Inputs

| Name                     | Type          | Default | Description                                  |
|--------------------------|---------------|---------|----------------------------------------------|
| `environment`            | string        | -       | Deployment environment (e.g., prod/dev).     |
| `vpc_cidr`               | string        | -       | CIDR block for the VPC.                      |
| `public_subnet_cidrs`    | list(string)  | -       | CIDR blocks for public subnets.              |
| `private_subnet_cidrs`   | list(string)  | -       | CIDR blocks for private subnets.             |
| `isolated_subnet_cidrs`  | list(string)  | -       | CIDR blocks for isolated subnets.            |
| `enable_nat_gateway`     | bool          | true    | Enables NAT Gateways for private subnets.    |
| `create_endpoints`       | bool          | true    | Creates VPC endpoints (S3, DynamoDB).        |
| `tags`                   | map(string)   | `{}`    | Tags applied to all resources.               |

## Outputs

| Output Name       | Description                       |
|-------------------|-----------------------------------|
| `vpc_id`          | ID of the created VPC.            |
| `public_subnets`  | IDs of public subnets.            |
| `private_subnets` | IDs of private subnets.           |
| `isolated_subnets`| IDs of isolated subnets.          |
| `nat_gateway_ids` | IDs of NAT Gateways.              |
| `endpoint_ids`    | IDs of created VPC endpoints.     |

## Examples

Typically invoked in prod Terraform before deploying compute workloads like Lambdas.

## Dependencies

- AWS Control Tower or configured AWS account.
- VirtualAgentics **Naming Conventions** and **Addressing Plan**.
- Terraform state backend setup.

## IAM/Security Considerations

- Subnets private unless explicitly public.
- NAT Gateway manages private subnet egress.

## Testing

- Verify creation in AWS console/CLI.
- Validate Terraform configuration.

## Versioning

Managed in `virtualagentics-iac` using semantic versioning. Phase 1 is stable draft.

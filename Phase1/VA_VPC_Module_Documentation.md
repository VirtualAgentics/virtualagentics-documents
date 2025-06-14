
---
title: "VA-VPC Terraform Module Documentation"
status: "Stable Draft"
version: "1.0"
date: "2025-06-14"
authors: "VirtualAgentics Infrastructure Team"
---

# Overview

The **VA-VPC** Terraform module establishes the foundational AWS Virtual Private Cloud (VPC) and subnet infrastructure for Phase 1 of the VirtualAgentics project. This module addresses the essential need for a secure, isolated, and structured networking environment within the AWS cloud, specifically tailored for VirtualAgentics' production workloads. The resulting VPC is the core network backbone that supports all agent-based activities and associated AWS resources deployed within the production environment.

# Scope/Purpose

This module provisions a VPC configured with a specified CIDR block (such as `10.0.0.0/16` for production) and organizes the network into a robust three-tier subnet architecture across multiple AWS Availability Zones (AZs):

- **Public subnets:** For components requiring internet-facing connectivity.
- **Private subnets:** For internal applications and services, equipped with NAT Gateways to allow secure internet egress.
- **Isolated subnets:** For highly secure backend resources with no internet connectivity.

The subnet CIDR allocations strictly adhere to the predefined VirtualAgentics addressing plan, ensuring non-overlapping, environment-specific IP spaces. Additionally, the module configures essential routing resources, including an Internet Gateway for public subnet connectivity and NAT Gateways for outbound internet access from private subnets. VPC endpoints for critical AWS services such as S3 and DynamoDB are provisioned to enable private service connectivity without exposing network traffic to the public internet.

# Assumptions/Constraints

- This module targets the production environment's core VPC in Phase 1. CIDR defaults for other environments (e.g., dev or staging) will differ per the addressing plan.
- Required inputs include:
  - `environment`: Used in resource naming.
  - `vpc_cidr`: Base CIDR block for the VPC.
  - Explicit CIDR ranges for public, private, and isolated subnets.
- The module assumes the AWS region used has at least three AZs available for subnet distribution.
- Default AWS resources like route tables or security groups are explicitly configured or replaced by this module to ensure compliance and security.

# Architecture/Diagram

The VPC is architecturally structured as follows:

- One VPC spanning multiple AZs.
- Three public subnets, each in a distinct AZ.
- Three private subnets, each paired with NAT Gateways for secure outbound traffic.
- Three isolated subnets for sensitive backend services.
- Internet Gateway attached for public subnet internet access.
- VPC Endpoints provisioned for secure, private AWS service access (S3, DynamoDB).

# Usage Example

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

# Inputs

| Name                     | Type          | Default | Description                                  |
|--------------------------|---------------|---------|----------------------------------------------|
| `environment`            | string        | -       | Deployment environment (e.g., prod, dev).    |
| `vpc_cidr`               | string        | -       | CIDR block for the VPC.                      |
| `public_subnet_cidrs`    | list(string)  | -       | CIDR blocks for public subnets.              |
| `private_subnet_cidrs`   | list(string)  | -       | CIDR blocks for private subnets.             |
| `isolated_subnet_cidrs`  | list(string)  | -       | CIDR blocks for isolated subnets.            |
| `enable_nat_gateway`     | bool          | true    | Enables NAT Gateways for private subnets.    |
| `create_endpoints`       | bool          | true    | Creates VPC endpoints (S3, DynamoDB).        |
| `tags`                   | map(string)   | `{}`    | Tags applied to all resources.               |

# Outputs

| Output Name      | Description                               |
|------------------|-------------------------------------------|
| `vpc_id`         | ID of the created VPC.                    |
| `public_subnets` | IDs of the public subnets.                |
| `private_subnets`| IDs of the private subnets.               |
| `isolated_subnets`| IDs of the isolated subnets.             |
| `nat_gateway_ids`| IDs of NAT Gateways created.              |
| `endpoint_ids`   | IDs of VPC endpoints created.             |

# Examples

This module would typically be invoked within the prod environment Terraform configurations before deploying compute workloads such as Lambda functions.

# Dependencies

- AWS Control Tower or AWS account setup.
- VirtualAgentics **Naming Conventions** and **Addressing Plan** documentation.
- Terraform state backend setup.

# IAM/Security Considerations

- Subnets are private unless explicitly configured as public.
- Outbound internet access from private subnets is managed securely through NAT Gateways.

# Testing

- Confirm VPC and subnets creation via AWS Console or CLI.
- Validate Terraform configuration.

# Versioning

This module is versioned in the `virtualagentics-iac` Git repository using semantic versioning.

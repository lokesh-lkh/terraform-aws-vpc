# Terraform AWS VPC Module

Production-ready Terraform module for provisioning a complete **AWS Virtual Private Cloud (VPC)** infrastructure with multi-AZ subnet tiers, NAT gateway, and optional VPC peering. Designed as the networking foundation for microservice deployments like **RoboShop**.

## Architecture

```
                    ┌─────────────────────────────────┐
                    │           VPC                    │
                    │        10.0.0.0/16              │
                    │                                  │
  Public Internet   │  ┌──────────────────────────────┤
  ──────────────────┼─►│  Public Subnets (2 AZs)      │
                    │  │  10.0.1.0/24  us-east-1a     │
                    │  │  10.0.2.0/24  us-east-1b     │
                    │  │  ┌──────┐                    │
                    │  │  │ NAT  │◄──┐                │
                    │  │  └──────┘   │                │
                    │  │             │                │
                    │  │ ├───────────┤                │
                    │  │ │ Private Subnets (2 AZs)    │
                    │  │ │  10.0.11.0/24 us-east-1a   │
                    │  │ │  10.0.12.0/24 us-east-1b   │
                    │  │ │  ┌──────┐ ┌──────┐         │
                    │  │ │  │App   │ │App   │          │
                    │  │ │  └──────┘ └──────┘          │
                    │  │ ├───────────┤                │
                    │  │ │ Database Subnets (2 AZs)    │
                    │  │ │  10.0.21.0/24 us-east-1a   │
                    │  │ │  10.0.22.0/24 us-east-1b   │
                    │  │ │  ┌──────┐ ┌──────┐         │
                    │  │ │  │ RDS  │ │ RDS │          │
                    │  │ │  └──────┘ └──────┘          │
                    │  │ └─────────────────────────────┘
                    │  └──────────────────────────────┘
                    └─────────────────────────────────┘
```

## Features

- ✅ **Multi-AZ VPC** with public, private, and database subnet tiers
- ✅ **Internet Gateway** for public subnet internet access
- ✅ **NAT Gateway** with Elastic IP for private subnet outbound traffic
- ✅ **Separate route tables** per subnet tier
- ✅ **Optional VPC Peering** to connect with default VPC
- ✅ **DB Subnet Group** pre-configured for RDS
- ✅ **Fully tagged** resources with project/environment prefix
- ✅ **Customizable CIDR blocks** and subnet ranges

## Prerequisites

- Terraform >= 1.0
- AWS Provider >= 4.0
- AWS credentials with VPC, EC2, and Route53 permissions
- Region with at least 2 Availability Zones

## Quick Start

```hcl
module "vpc" {
  source = "git::https://github.com/practice-org-io/terraform-aws-vpc.git?ref=main"

  project       = "roboshop"
  environment   = "dev"
  vpc_cidr      = "10.0.0.0/16"

  # Optional: customize subnet CIDRs
  public_subnet_cidrs   = ["10.0.1.0/24", "10.0.2.0/24"]
  private_subnet_cidrs  = ["10.0.11.0/24", "10.0.12.0/24"]
  database_subnet_cidrs = ["10.0.21.0/24", "10.0.22.0/24"]

  # Enable VPC peering with default VPC (optional)
  is_peering_required = false
}
```

## Resources Created

| Resource | Description |
|----------|-------------|
| `aws_vpc.main` | Main VPC with DNS support enabled |
| `aws_internet_gateway.main` | IGW for public internet access |
| `aws_subnet.public[*]` | Public subnets in each AZ |
| `aws_subnet.private[*]` | Private subnets in each AZ |
| `aws_subnet.database[*]` | Database subnets in each AZ |
| `aws_nat_gateway.main` | NAT gateway for private egress |
| `aws_eip.nat` | Elastic IP for NAT gateway |
| `aws_route_table.*` | Route tables per subnet tier |
| `aws_route.*` | Routes for internet/NAT/peering |
| `aws_route_table_association.*` | Associate route tables to subnets |
| `aws_db_subnet_group.roboshop` | DB subnet group for RDS |
| `aws_vpc_peering_connection` | VPC peering (optional) |

## Variables

| Name | Type | Default | Required | Description |
|------|------|---------|:--------:|-------------|
| `project` | `string` | — | ✅ | Project name used for resource naming |
| `environment` | `string` | — | ✅ | Environment: `dev`, `qa`, `uat`, or `prod` |
| `vpc_cidr` | `string` | `10.0.0.0/16` | ❌ | CIDR block for the VPC |
| `public_subnet_cidrs` | `list(string)` | `["10.0.1.0/24", "10.0.2.0/24"]` | ❌ | CIDR blocks for public subnets |
| `private_subnet_cidrs` | `list(string)` | `["10.0.11.0/24", "10.0.12.0/24"]` | ❌ | CIDR blocks for private subnets |
| `database_subnet_cidrs` | `list(string)` | `["10.0.21.0/24", "10.0.22.0/24"]` | ❌ | CIDR blocks for database subnets |
| `is_peering_required` | `bool` | `false` | ❌ | Enable VPC peering with default VPC |
| `vpc_tags` | `map(string)` | `{}` | ❌ | Additional tags for VPC |
| `igw_tags` | `map(string)` | `{}` | ❌ | Additional tags for IGW |
| `public_subnet_tags` | `map(string)` | `{}` | ❌ | Additional tags for public subnets |
| `private_subnet_tags` | `map(string)` | `{}` | ❌ | Additional tags for private subnets |
| `database_subnet_tags` | `map(string)` | `{}` | ❌ | Additional tags for database subnets |
| `eip_tags` | `map(string)` | `{}` | ❌ | Additional tags for Elastic IP |
| `nat_gateway_tags` | `map(string)` | `{}` | ❌ | Additional tags for NAT gateway |

## Outputs

| Output | Description |
|--------|-------------|
| `vpc_id` | The ID of the created VPC |
| `public_subnet_ids` | List of public subnet IDs |
| `private_subnet_ids` | List of private subnet IDs |
| `database_subnet_ids` | List of database subnet IDs |
| `database_subnet_group_name` | Name of the DB subnet group |
| `azs_info` | Available availability zones in the region |

## File Structure

```
terraform-aws-vpc/
├── main.tf         # Core VPC resources (subnets, gateways, routes)
├── variables.tf    # Input variable definitions
├── outputs.tf      # Module outputs
├── locals.tf       # Local values and tag merging
├── data.tf         # Data sources (availability zones, default VPC)
├── peering.tf      # VPC peering resources (optional)
└── README.md       # This file
```

## Naming Convention

All resources follow the pattern:

```
{project}-{environment}-{tier}-{availability-zone}
```

**Examples:**
- `roboshop-dev-public-us-east-1a`
- `roboshop-dev-private-us-east-1b`
- `roboshop-dev-database-us-east-1a`

## Related Repos

- [terraform-aws-instance](../terraform-aws-instance) — EC2 provisioning
- [terraform-roboshop-component](../terraform-roboshop-component) — Full RoboShop infra
- [roboshop-infra-dev](../roboshop-infra-dev) — Complete dev environment

## License

MIT

# Part 1 – Network Stack (CloudFormation)

---

## Objective

Design and deploy a secure, highly available VPC architecture to serve as the foundational network layer for an Amazon EKS multi-tier environment.

This network stack provides:

- Public subnets for internet-facing resources  
- Private subnets for application and worker nodes  
- Controlled outbound internet access via NAT  
- Multi-AZ high availability design  

All resources were provisioned using **Infrastructure as Code (CloudFormation)**.

---

## Architecture Overview

The VPC architecture includes:

- 1 VPC  
- 2 Public Subnets (across different AZs)  
- 2 Private Subnets (across different AZs)  
- 1 Internet Gateway  
- 1 NAT Gateway  
- Separate Route Tables for public and private subnets  

This layout ensures proper network segmentation between internet-facing and internal resources.

---

## Architecture Design Decisions

### High Availability

- Subnets are distributed across multiple Availability Zones.
- Public and private resources are deployed redundantly.
- Designed to tolerate an AZ outage without complete service disruption.

### Security & Network Segmentation

- Worker nodes and application workloads are placed in **private subnets**.
- Private subnets do not allow direct inbound internet access.
- Internet Gateway is attached only to public subnets.
- NAT Gateway enables outbound internet access for private resources.
- Route tables are separated to enforce traffic control boundaries.

This design follows production-level AWS networking best practices.

---

## CIDR Strategy

| Resource | CIDR Block |
|----------|------------|
| VPC | 10.0.0.0/16 |
| Public Subnet A | 10.0.1.0/24 |
| Public Subnet B | 10.0.2.0/24 |
| Private Subnet A | 10.0.11.0/24 |
| Private Subnet B | 10.0.12.0/24 |

Subnets were segmented to logically separate public-facing infrastructure from internal application resources.

---

## CloudFormation Implementation

The VPC was provisioned using a parameterized CloudFormation template:

- Reusable parameters  
- Tagged resources  
- Explicit subnet and route table associations  
- Internet and NAT gateway configuration  
- Outputs exported for use in later stacks (EKS Cluster & Node Groups)

**Template file:**

vpc_template.yaml



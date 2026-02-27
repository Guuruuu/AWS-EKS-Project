# Part 2 – EKS Cluster (Control Plane)

---

## Objective

Provision a managed Amazon EKS control plane using CloudFormation within the previously created VPC network stack.

This milestone establishes the Kubernetes control plane that will manage worker nodes and application workloads.

---

## Architecture Overview

The EKS control plane was deployed with the following configuration:

- Managed Amazon EKS Cluster
- Integrated with custom VPC (from Part 1)
- Private subnet deployment
- Cluster security group configuration
- IAM role for EKS service

The cluster spans multiple Availability Zones for high availability and resilience.

---

## Architecture Design Decisions

### Private Subnet Deployment

The EKS control plane is integrated with private subnets created in Milestone 1.

This ensures:

- No direct public exposure of worker nodes
- Controlled API access
- Reduced attack surface
- Production-aligned security design

---

### IAM Role Separation

A dedicated IAM role was created for the EKS control plane:

- Trusted by `eks.amazonaws.com`
- Allows EKS to manage cluster resources
- Follows principle of least privilege

This separates control plane permissions from worker node permissions.

---

### Security Group Configuration

Cluster security groups were configured to:

- Allow HTTPS (port 443) communication
- Enable communication between worker nodes and control plane
- Restrict unnecessary inbound access

This ensures secure Kubernetes API communication.

---

## CloudFormation Implementation

The EKS cluster was provisioned using a parameterized CloudFormation template.

Key components defined in the template:

- `AWS::EKS::Cluster`
- IAM Role for EKS
- VPC and subnet references (imported from Part 1)
- Security group configuration
- Cluster logging configuration

Template file:

eks_cluster_template.yaml

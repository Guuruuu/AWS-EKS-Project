# Part 3 – Worker Nodes (Node Group)

---

## Objective

Provision and attach EC2 worker nodes to the Amazon EKS control plane using CloudFormation.

This milestone establishes the compute layer of the Kubernetes cluster, allowing workloads and pods to be scheduled.

---

## Architecture Overview

The worker node configuration includes:

- EC2 instances deployed in private subnets
- Auto Scaling Group for node management
- IAM role for worker nodes
- Security group configuration for cluster communication
- Integration with EKS control plane

Worker nodes span multiple Availability Zones for high availability.

---

## Architecture Design Decisions

### Private Subnet Placement

Worker nodes are deployed in private subnets to:

- Prevent direct internet exposure
- Reduce attack surface
- Enforce secure outbound-only access via NAT Gateway
- Follow production security best practices

---

### IAM Role for Worker Nodes

A dedicated IAM role was created for the node group.

This role includes policies such as:

- `AmazonEKSWorkerNodePolicy`
- `AmazonEC2ContainerRegistryReadOnly`
- `AmazonEKS_CNI_Policy`

This allows worker nodes to:

- Register with the EKS control plane
- Pull container images from ECR
- Configure networking via the VPC CNI plugin

Separation of IAM roles ensures control plane and node permissions remain isolated.

---

### Kubernetes Node Registration

Each EC2 instance runs:

- kubelet
- container runtime
- AWS VPC CNI plugin

The kubelet allows nodes to:

- Authenticate with the control plane
- Join the cluster
- Receive pod scheduling instructions

---

### High Availability

Nodes are distributed across private subnets in multiple Availability Zones to:

- Improve cluster resilience
- Prevent single-AZ dependency
- Support fault tolerance

---

## CloudFormation Implementation

Worker nodes were provisioned using a CloudFormation template that defines:

- Launch template / EC2 configuration
- Auto Scaling Group
- Node IAM role
- Security group rules
- Subnet associations

The template integrates with the EKS cluster created in Part 2.

Template file:

worker_nodes_template.yaml

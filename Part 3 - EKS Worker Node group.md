Overview

Part 3 focuses on provisioning the EKS Worker Node Group using AWS CloudFormation.
This milestone builds on the VPC (Part 1) and EKS Control Plane (Part 2) by deploying the compute layer that will run Kubernetes workloads.

The worker nodes are deployed inside the private subnets of the VPC and are associated with a dedicated IAM role, scaling configuration, labels, and subnet configuration for high availability.

Objectives Completed

Created an IAM Role for EKS worker nodes

Attached required AWS-managed policies

Created an EKS NodeGroup resource

Imported values from previous stacks (ClusterName, Private Subnets)

Enabled multi-AZ node distribution

Added meaningful node labels

Configured NodeGroup scaling

Verified proper CloudFormation stack linking via Outputs/ImportValue

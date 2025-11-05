# Milestone 1 – Creating an EKS Cluster

##  Objective
To create an Amazon Elastic Kubernetes Service (EKS) cluster inside the custom VPC created earlier.  
This cluster will serve as the control plane that manages worker nodes and workloads within private subnets.

---

##  Step 1 – Purpose of the EKS Cluster
The EKS cluster is the **control plane** for Kubernetes in AWS.  
It manages the scheduling of pods, monitoring of nodes, and communication between the Kubernetes API and worker nodes.  
We use **CloudFormation** to automate its deployment rather than configuring it manually through the console.

---

##  Step 2 – Cluster Configuration
The cluster is created using the CloudFormation resource **`AWS::EKS::Cluster`**.  
Key parameters defined include:

- **Name** – dynamically generated with the environment name (e.g., `mtier-eks-cluster`).  
- **Version** – specifies the Kubernetes version (`1.33`).  
- **RoleArn** – IAM role that gives EKS permission to interact with AWS resources like EC2 and CloudWatch.  
- **ResourcesVpcConfig** – defines networking configuration:
  - **SubnetIds** – the private subnets (`AppSubnetA` and `AppSubnetB`) where EKS control plane ENIs (Elastic Network Interfaces) will be created.  
  - **EndpointPublicAccess: false** – disables public endpoint access for security.  
  - **EndpointPrivateAccess: true** – enables access only through private network (via SSM or Bastion).

---

##  Step 3 – CloudFormation Snippet

```yaml
# Creating an EKS cluster
EksCluster:
  Type: AWS::EKS::Cluster
  Properties:
    Name: !Sub "${EnvName}-eks-cluster"
    Version: 1.33
    RoleArn: arn:aws:iam::339298461301:role/AmazonEKSAutoClusterRole
    ResourcesVpcConfig:
      SubnetIds:
        - !Ref AppSubnetA
        - !Ref AppSubnetB
      EndpointPublicAccess: false
      EndpointPrivateAccess: true

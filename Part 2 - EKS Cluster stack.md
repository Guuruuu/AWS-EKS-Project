## Milestone 2 — Secure EKS Control Plane Deployment (CloudFormation)

### Objective  
Deploy the Amazon EKS control plane and admin access layer using CloudFormation, with:
- A private-only EKS API endpoint
- An SSM-based bastion host (no SSH)
- IAM roles and security groups fully managed as code

---

## Infrastructure Built in This Milestone

### 1. SSM Bastion Host Security Group

- No inbound rules (all access via SSM Session Manager)
- Outbound allowed so the bastion can reach:
  - SSM endpoints
  - EKS API server
  - Internet via NAT

```yaml
SsmBastionHostSG:
  Type: AWS::EC2::SecurityGroup
  Properties:
    GroupDescription: SSM Bastion host (no inbound; outbound via NAT)
    VpcId: !ImportValue mtier:VpcId
    SecurityGroupIngress: []  # no inbound traffic
    SecurityGroupEgress:
      - IpProtocol: -1
        CidrIp: 0.0.0.0/0  # allow all outbound
    Tags:
      - Key: Name
        Value: ssm-bastion-sg

2. IAM Role for Bastion + Inline EKS Describe Permissions

Allows the EC2 instance to:

Register with SSM (AmazonSSMManagedInstanceCore)

Call eks:DescribeCluster so aws eks update-kubeconfig works

SsmIamRole:
  Type: AWS::IAM::Role
  Properties:
    RoleName: mtier-ssm-role
    Description: IAM role that allows you to use SSM to communicate with the Kubernetes cluster
    AssumeRolePolicyDocument:
      Version: '2012-10-17'
      Statement:
        - Effect: Allow
          Principal:
            Service: ec2.amazonaws.com
          Action: sts:AssumeRole
    ManagedPolicyArns:
      - arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore
    Policies:
      - PolicyName: mtier-eks-describecluster
        PolicyDocument:
          Version: '2012-10-17'
          Statement:
            - Effect: Allow
              Action:
                - eks:DescribeCluster
              Resource: "*"
    Tags:
      - Key: Name
        Value: mtier-ssm-role

3. Instance Profile + Bastion EC2 Instance

Private subnet only (no public IP)

Uses SSM role above

Installs kubectl in UserData

SsmInstanceProfile:
  Type: AWS::IAM::InstanceProfile
  Properties:
    Roles:
      - !Ref SsmIamRole

SsmBastionHostInstance:
  Type: AWS::EC2::Instance
  Properties:
    ImageId: ami-xxxxxxxxxxxxxxxxx        # Amazon Linux 2/2023 AMI for your region
    InstanceType: !Ref BastionInstanceType
    IamInstanceProfile: !Ref SsmInstanceProfile
    SubnetId: !ImportValue mtier:AppSubnetA
    SecurityGroupIds:
      - !Ref SsmBastionHostSG
    Tags:
      - Key: Name
        Value: mtier-bastion-host
    UserData:
      Fn::Base64: !Sub |
        #!/bin/bash
        # Install kubectl (latest stable)
        curl -LO "https://dl.k8s.io/release/$(curl -Ls https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
        chmod +x kubectl
        mv -f kubectl /usr/local/bin/kubectl

4. EKS Control Plane IAM Role

Trusted service: eks.amazonaws.com

Managed policy: AmazonEKSClusterPolicy

Used by the EKS control plane to interact with AWS resources

EksIamRole:
  Type: AWS::IAM::Role
  Properties:
    RoleName: eks-role
    Description: IAM role for EKS control plane
    AssumeRolePolicyDocument:
      Version: "2012-10-17"
      Statement:
        - Effect: Allow
          Principal:
            Service: eks.amazonaws.com
          Action: sts:AssumeRole
    Path: "/"
    ManagedPolicyArns:
      - arn:aws:iam::aws:policy/AmazonEKSClusterPolicy

5. EKS Cluster Security Group

Allows HTTPS (443) from the bastion host SG to the API server

Allows egress inside the VPC CIDR so the control plane can communicate with cluster components

EksClusterSG:
  Type: AWS::EC2::SecurityGroup
  Properties:
    GroupDescription: Eks security group (Cluster)
    VpcId: !ImportValue mtier:VpcId
    SecurityGroupIngress:
      - IpProtocol: tcp
        FromPort: 443
        ToPort: 443
        SourceSecurityGroupId: !Ref SsmBastionHostSG
    SecurityGroupEgress:
      - IpProtocol: -1
        CidrIp: !ImportValue mtier:VpcCidr
    Tags:
      - Key: Name
        Value: !Sub "${EnvName}-eks-cluster-sg"

6. EKS Cluster Resource

Uses:

VPC + subnet IDs from the network stack (Milestone 1)

The EKS IAM role

The EKS Cluster SG

Private endpoint only (no public API exposure)

EksCluster:
  Type: AWS::EKS::Cluster
  Properties:
    Name: !Ref ClusterName
    Version: !Ref ClusterVersion
    RoleArn: !GetAtt EksIamRole.Arn
    ResourcesVpcConfig:
      VpcId: !ImportValue mtier:VpcId
      SubnetIds:
        - !ImportValue mtier:AppSubnetA
        - !ImportValue mtier:AppSubnetB
      SecurityGroupIds:
        - !Ref EksClusterSG
      EndpointPublicAccess: false
      EndpointPrivateAccess: true

Security & Networking Behavior

Bastion Host

No inbound rules; access only via AWS Systems Manager Session Manager

Can reach the Kubernetes API server on TCP 443

EKS Control Plane

Runs as a managed service, with ENIs attached to the private subnets

Secured by EksClusterSG

Only accepts API traffic from the Bastion SG at this stage

VPC Integration

All VPC/subnet IDs and CIDR values are imported from the network stack (Milestone 1) using !ImportValue.


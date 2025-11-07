
#  Milestone 1 — SSM Bastion Host Setup
*Secure access to private AWS EKS and RDS resources via AWS Systems Manager (SSM)*  

---

### Objective  
Create a secure **bastion host** inside the **private application subnet** that allows management of private AWS resources (like the EKS cluster) without exposing them to the internet.  
Access is handled through **AWS Systems Manager Session Manager**, not SSH.

---

###  Architecture Overview  
The SSM bastion host serves as a secure jump point to reach internal resources.  
It sits in a **private AppSubnet**, uses the **NAT Gateway** for outbound updates, and communicates with AWS SSM over HTTPS (port 443).  
AWS Console / CloudShell

↓

AWS SSM Session Manager

↓

Bastion Host (Private Subnet)

↓

EKS Cluster (Private Subnets)

No public IP needed  
No SSH keys required  
All traffic is auditable via AWS SSM logs  

---

### Components Created  

#### Security Group – `SsmBastionHostSG`
```yaml
SsmBastionHostSG:
  Type: AWS::EC2::SecurityGroup
  Properties:
    GroupDescription: SSM bastion host (no inbound; outbound via NAT)
    GroupName: !Sub "${EnvName}-bastion-sg"
    VpcId: !Ref VPC
    SecurityGroupIngress: []    # no inbound traffic
    SecurityGroupEgress:
      - IpProtocol: -1
        CidrIp: 0.0.0.0/0      # allow all outbound traffic through NAT
    Tags:
      - Key: Name
        Value: !Sub "${EnvName}-bastion-sg"
Restricts inbound traffic while allowing outbound system updates and EKS API communication.

IAM Role – SsmIamRole

SsmIamRole:
  Type: AWS::IAM::Role
  Properties:
    RoleName: !Sub "${EnvName}-ssm-role"
    AssumeRolePolicyDocument:
      Version: "2012-10-17"
      Statement:
        - Effect: Allow
          Principal:
            Service: ec2.amazonaws.com
          Action: sts:AssumeRole
    ManagedPolicyArns:
      - arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore
    Tags:
      - Key: Name
        Value: !Sub "${EnvName}-ssm-role"
Allows the EC2 instance to use SSM Session Manager instead of SSH.

Instance Profile – SsmInstanceProfile

SsmInstanceProfile:
  Type: AWS::IAM::InstanceProfile
  Properties:
    Roles:
      - !Ref SsmIamRole
Binds the IAM role to the EC2 instance so it inherits SSM permissions.

EC2 Instance – SsmBastionHostInstance

SsmBastionHostInstance:
  Type: AWS::EC2::Instance
  Properties:
    ImageId: ami-0157af9aea2eef346
    InstanceType: !Ref BastionInstanceType
    IamInstanceProfile: !Ref SsmInstanceProfile
    SubnetId: !Ref AppSubnetA
    SecurityGroupIds:
      - !Ref SsmBastionHostSG
    Tags:
      - Key: Name
        Value: !Sub "${EnvName}-bastion-host"
    UserData:
      Fn::Base64: !Sub |
        #!/bin/bash
        set -e
        # Install kubectl (latest stable)
        curl -LO https://dl.k8s.io/release/$(curl -Ls https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl
        install -m 0755 kubectl /usr/local/bin/kubectl
        rm -f kubectl

**Launches a private EC2 bastion host with kubectl preinstalled for EKS management.**

Access Method

Connect securely via Session Manager:

aws ssm start-session --target <i-xxxxxxxxxxxxxxx>
or
EC2 Console → Instance → Connect → Session Manager tab → Connect

Once connected:

kubectl get nodes
kubectl get pods



**Key Benefits**

-No public exposure or SSH keys

-Auditable access via AWS SSM

-Fully contained in private subnet

-Can manage EKS and RDS internally





# Milestone 1 – Foundations

## Objective
## Architecture Design Decisions

- Created 2 public subnets across different AZs for high availability.
- Created 2 private subnets for EKS worker nodes.
- NAT Gateway placed in public subnet to allow private subnet internet access.
- Route tables separated for public and private traffic control.

---

## Step 1 – Create Custom VPC (CloudFormation)

I created a custom VPC using a CloudFormation template named **vpc.yaml**.  
This template provisions:
- 2 public subnets  
- 2 private subnets  
- 1 Internet Gateway  
- 1 NAT Gateway  
- Route tables for each subnet type  

### YAML Snippet
```yaml
Parameters:
  EnvName:
    Type: String
    Default: mtier

Resources:
  VPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      Tags:
        - Key: Name
          Value: !Sub "${EnvName}-vpc"



# Milestone 1 – Foundations

## Objective
Set up the base AWS infrastructure: custom VPC, EKS cluster, and core add-ons.

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



# Program 1: Create an EC2 Instance Using AWS CloudFormation

## Aim

To create an Amazon Linux 2023 EC2 instance using an AWS CloudFormation YAML template.

## Prerequisites

- An AWS account
- AWS CLI installed on Windows
- AWS CLI configured with valid credentials
- A default VPC in the required AWS Region
- An existing EC2 Key Pair
- AWS Region: `us-east-1`

---

## Step 1: Check AWS CLI Configuration

Open PowerShell and run:

```powershell
aws configure list
```

Verify that the AWS Region is set to:

```text
us-east-1
```

Test the credentials:

```powershell
aws sts get-caller-identity
```

If the command returns your AWS account information, the AWS CLI is working correctly.

---

## Step 2: Create a Working Directory

Create a directory for the CloudFormation files:

```powershell
mkdir C:\Users\Ajay Kumar Badhan\docker_practise\aws_cloud_formation
```

Move into the directory:

```powershell
cd C:\Users\Ajay Kumar Badhan\docker_practise\aws_cloud_formation
```

---

## Step 3: Create the CloudFormation YAML File

Create a file named:

```text
ec2-instance.yaml
```

You can open it with:

```powershell
notepad ec2-instance.yaml
```

Paste the following template:

```yaml
AWSTemplateFormatVersion: '2010-09-09'

Description: Create a simple Amazon Linux EC2 instance

Parameters:

  InstanceType:
    Type: String
    Default: t3.micro
    Description: EC2 instance type

  KeyName:
    Type: AWS::EC2::KeyPair::KeyName
    Description: Name of an existing EC2 KeyPair

Resources:

  MyEC2Instance:
    Type: AWS::EC2::Instance

    Properties:
      InstanceType: !Ref InstanceType

      ImageId: '{{resolve:ssm:/aws/service/ami-amazon-linux-latest/al2023-ami-kernel-default-x86_64}}'

      KeyName: !Ref KeyName

      Tags:
        - Key: Name
          Value: CloudFormation-EC2

Outputs:

  InstanceId:
    Description: EC2 Instance ID
    Value: !Ref MyEC2Instance

  PublicIP:
    Description: Public IP address of the EC2 instance
    Value: !GetAtt MyEC2Instance.PublicIp
```

Save the file and close Notepad.

---

## Step 4: Validate the YAML Template

Run:

```powershell
aws cloudformation validate-template --template-body file://ec2-instance.yaml
```

If the template is valid, AWS will return information about the template parameters.

> **Important:** Use `file://ec2-instance.yaml`, not just `ec2-instance.yaml`.

---

## Step 5: Create the CloudFormation Stack

Run:

```powershell
aws cloudformation create-stack `
  --stack-name MyFirstEC2Stack `
  --template-body file://ec2-instance.yaml `
  --parameters ParameterKey=KeyName,ParameterValue=YOUR_KEY_NAME
```

Replace `YOUR_KEY_NAME` with the name of your existing EC2 Key Pair.

For example:

```powershell
aws cloudformation create-stack `
  --stack-name MyFirstEC2Stack `
  --template-body file://ec2-instance.yaml `
  --parameters ParameterKey=KeyName,ParameterValue=mykey
```

---

## Step 6: Check Stack Status

Run:

```powershell
aws cloudformation describe-stacks --stack-name MyFirstEC2Stack
```

You can also monitor the events:

```powershell
aws cloudformation describe-stack-events --stack-name MyFirstEC2Stack
```

Wait until the stack reaches:

```text
CREATE_COMPLETE
```

---

## Step 7: Verify the EC2 Instance

Open:

**AWS Console → EC2 → Instances**

Look for:

```text
CloudFormation-EC2
```

Verify:

- Instance state: Running
- Instance type: `t3.micro`
- AMI: Amazon Linux 2023
- Key pair: Your selected key pair

---

## Step 8: Check CloudFormation Outputs

Run:

```powershell
aws cloudformation describe-stacks `
  --stack-name MyFirstEC2Stack `
  --query "Stacks[0].Outputs"
```

You should see outputs such as:

```text
InstanceId
PublicIP
```

---

## Step 9: Delete the Stack When Finished

To remove the EC2 instance created by this lab:

```powershell
aws cloudformation delete-stack --stack-name MyFirstEC2Stack
```

Check the deletion:

```powershell
aws cloudformation describe-stacks --stack-name MyFirstEC2Stack
```

After deletion, the stack and its EC2 instance will no longer exist.

---

## Expected Result

An Amazon Linux 2023 EC2 instance is successfully created using an AWS CloudFormation YAML template.

The CloudFormation stack should show:

```text
CREATE_COMPLETE
```

---

## Key CloudFormation Concepts Learned

| Concept | Purpose |
|---|---|
| `AWS::EC2::Instance` | Creates an EC2 instance |
| `Parameters` | Accepts values such as instance type and key pair |
| `!Ref` | References a parameter/resource |
| `!GetAtt` | Retrieves a resource attribute |
| `Outputs` | Displays useful information after deployment |
| SSM AMI parameter | Dynamically resolves the latest Amazon Linux 2023 AMI |
| CloudFormation Stack | Manages the infrastructure as a single unit |

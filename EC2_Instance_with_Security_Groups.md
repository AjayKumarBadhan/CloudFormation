# Program 2: Update an Existing CloudFormation Stack by Adding a Security Group

## Aim

To update an existing AWS CloudFormation stack by adding a Security Group and attaching it to an existing EC2 instance.

This program continues from **Program 1**.

## Prerequisites

- Program 1 should have been completed successfully.
- The CloudFormation stack should already exist.
- Stack name:

```text
MyFirstEC2Stack
```

- AWS CLI should be configured with valid credentials.
- A default VPC should exist in `us-east-1`.

---

## Step 1: Verify the Existing Stack

Run:

```powershell
aws cloudformation describe-stacks --stack-name MyFirstEC2Stack
```

Check that the stack exists.

You can also check its status:

```powershell
aws cloudformation describe-stacks `
  --stack-name MyFirstEC2Stack `
  --query "Stacks[0].StackStatus"
```

The stack should normally be in a state such as:

```text
CREATE_COMPLETE
```

---

## Step 2: Open the Existing YAML File

Move to your CloudFormation directory:

```powershell
cd C:\Users\Ajay Kumar Badhan\docker_practise\aws_cloud_formation
```

Open the existing template:

```powershell
notepad ec2-instance.yaml
```

---

## Step 3: Update the YAML Template

Replace the previous template with the following:

```yaml
AWSTemplateFormatVersion: '2010-09-09'

Description: Create an Amazon Linux EC2 instance with a Security Group

Parameters:

  InstanceType:
    Type: String
    Default: t3.micro
    Description: EC2 instance type

  KeyName:
    Type: AWS::EC2::KeyPair::KeyName
    Description: Name of an existing EC2 KeyPair

Resources:

  MySecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Allow SSH access to EC2
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0
      Tags:
        - Key: Name
          Value: CloudFormation-SecurityGroup

  MyEC2Instance:
    Type: AWS::EC2::Instance

    Properties:
      InstanceType: !Ref InstanceType

      ImageId: '{{resolve:ssm:/aws/service/ami-amazon-linux-latest/al2023-ami-kernel-default-x86_64}}'

      KeyName: !Ref KeyName

      SecurityGroupIds:
        - !GetAtt MySecurityGroup.GroupId

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

  SecurityGroupId:
    Description: Security Group ID
    Value: !GetAtt MySecurityGroup.GroupId
```

Save the file:

```text
Ctrl + S
```

Then close Notepad.

---

## Step 4: Understand What Was Added

The original template contained only the EC2 resource:

```yaml
MyEC2Instance:
  Type: AWS::EC2::Instance
```

The updated template adds a new resource:

```yaml
MySecurityGroup:
  Type: AWS::EC2::SecurityGroup
```

The Security Group allows SSH traffic:

```yaml
SecurityGroupIngress:
  - IpProtocol: tcp
    FromPort: 22
    ToPort: 22
    CidrIp: 0.0.0.0/0
```

The Security Group is attached to the EC2 instance using:

```yaml
SecurityGroupIds:
  - !GetAtt MySecurityGroup.GroupId
```

Therefore, CloudFormation manages both resources.

---

## Step 5: Validate the Updated Template

Run:

```powershell
aws cloudformation validate-template --template-body file://ec2-instance.yaml
```

If successful, the template is syntactically valid.

---

## Step 6: Update the Existing Stack

**Do not create a new stack.**

Use the existing stack name:

```text
MyFirstEC2Stack
```

Run:

```powershell
aws cloudformation update-stack `
  --stack-name MyFirstEC2Stack `
  --template-body file://ec2-instance.yaml `
  --parameters ParameterKey=KeyName,ParameterValue=YOUR_KEY_NAME
```

Replace `YOUR_KEY_NAME` with your actual EC2 Key Pair name.

For example:

```powershell
aws cloudformation update-stack `
  --stack-name MyFirstEC2Stack `
  --template-body file://ec2-instance.yaml `
  --parameters ParameterKey=KeyName,ParameterValue=mykey
```

---

## Step 7: Monitor the Stack Update

Run:

```powershell
aws cloudformation describe-stack-events --stack-name MyFirstEC2Stack
```

Look for:

```text
UPDATE_IN_PROGRESS
```

and finally:

```text
UPDATE_COMPLETE
```

You can also check the current status:

```powershell
aws cloudformation describe-stacks `
  --stack-name MyFirstEC2Stack `
  --query "Stacks[0].StackStatus"
```

---

## Step 8: Verify the Security Group

Open:

**AWS Console → EC2 → Instances → CloudFormation-EC2**

Select the instance.

Under the **Security** section, verify that a Security Group is attached.

The Security Group should have a name similar to:

```text
CloudFormation-SecurityGroup
```

---

## Step 9: Verify the Inbound Rule

Open the Security Group and select:

**Inbound rules**

You should see:

| Type | Protocol | Port | Source |
|---|---|---:|---|
| SSH | TCP | 22 | 0.0.0.0/0 |

This demonstrates how CloudFormation can create and attach networking/security resources to an EC2 instance.

---

## Security Warning

The lab uses:

```yaml
CidrIp: 0.0.0.0/0
```

This allows SSH connections from any IPv4 address on the Internet.

For a real environment, restrict SSH to your own public IP:

```yaml
CidrIp: YOUR_PUBLIC_IP/32
```

For example:

```yaml
CidrIp: 203.0.113.10/32
```

Do not use an example IP as your actual rule; replace it with your real public IP.

---

## Step 10: Check CloudFormation Outputs

Run:

```powershell
aws cloudformation describe-stacks `
  --stack-name MyFirstEC2Stack `
  --query "Stacks[0].Outputs"
```

The outputs should now include:

```text
InstanceId
PublicIP
SecurityGroupId
```

---

## Step 11: View Resources Managed by the Stack

Run:

```powershell
aws cloudformation list-stack-resources `
  --stack-name MyFirstEC2Stack
```

You should now see resources similar to:

```text
MyEC2Instance
MySecurityGroup
```

This demonstrates that both resources are managed by the same CloudFormation stack.

---

## Step 12: Delete the Stack When Finished

When the lab is complete:

```powershell
aws cloudformation delete-stack --stack-name MyFirstEC2Stack
```

CloudFormation will remove the resources that it created as part of the stack, including the EC2 instance and Security Group.

---

## Expected Result

The existing CloudFormation stack is successfully updated.

Before the update:

```text
CloudFormation Stack
        |
        └── EC2 Instance
```

After the update:

```text
CloudFormation Stack
        |
        ├── EC2 Instance
        |
        └── Security Group
                 |
                 └── SSH Port 22
```

The final stack status should be:

```text
UPDATE_COMPLETE
```

---

## Key Concepts Learned

| Concept | Purpose |
|---|---|
| `AWS::EC2::SecurityGroup` | Creates an EC2 Security Group |
| `SecurityGroupIngress` | Defines inbound traffic rules |
| `SecurityGroupIds` | Associates a Security Group with EC2 |
| `!GetAtt` | Retrieves the Security Group ID |
| `update-stack` | Updates an existing CloudFormation stack |
| `UPDATE_COMPLETE` | Indicates a successful stack update |
| Stack resources | Shows resources managed by CloudFormation |

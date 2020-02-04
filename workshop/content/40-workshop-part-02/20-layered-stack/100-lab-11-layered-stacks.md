---
title: 'Lab 11: Layered Stacks'
date: 2019-11-13T16:52:42Z
weight: 100
---

## Introduction

In the previous lab, we saw how we use the `Outputs` section and the `Fn::GetAtt` function to pass values from a child stack to parent stack. This enabled us to have dedicated templates for a VPC and an IAM role.
As we mentioned previously this gives us the ability to create **templates** that can be re-used. However what about if we want to re-use **stacks**?

For example, you may have plans for many different workloads deployed with many different templates but every EC2 instance is expected to enable Systems Manager Session Manager access to every EC2 Instance. Similarly, you may wish to deploy a VPC via one stack and then use it with multiple future stacks and workloads. Achieving this one-many relationship is not possible in a Nested Stack scenario. This is where Layered Stacks come in.

We use [Exports](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-stack-exports.html) to create global variables that can be [Imported](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-importvalue.html) into any CloudFormation stack.

## Lab Overview

In this lab, you will build:

1. **The VPC stack**. This contains the same simple VPC template used in the previous lab but with `Export` added to the `Outputs`.
1. **The IAM instance role stack**. This contains the same IAM instance role used in the previous lab but with `Export` added to the `Outputs`.
1. **The EC2 stack**. This contains the EC2 instance you have defined in previous labs but will make use of the `Fn::ImportValue` function.

Here is a diagram showing the hierarchy of layered stacks.

![layered-stack-hierarchy](../layered-stack-hierarchy.png)

This diagram represents the high-level overview of the infrastructure that will be deployed:

![layered-stack-architecture](../ls-architecture.png)

You will find the working directory in `code/70-layered-stack/01-working directory`. In the rest of this lab, you should add your code to the templates here.

You can find the working solutions in `code/70-layered-stack/02-solution`. You can reference this against your code.

**Let's start..**

### Create VPC Stack

The VPC template has been created for you. It is titled `vpc.yaml`. This template will create VPC stack with 2 Public Subnets, an Internet Gateway, and Route tables.

#### 1. Prepare the VPC template

{{% notice note %}} 
All of the files referenced in this lab can be found within `code/70-layered-stack/01-working directory`
{{% /notice %}}

If you look in the file `vpc.yaml` file, you will notice that there are some outputs in the _Outputs_ section of the template.
You will now add exports to each of these so that we can consume them from other CloudFormation stacks.

Add the highlighted lines shown below to your template file.
```yaml {hl_lines=[4,5,9,10,14,15]}
Outputs:
  VpcId:
    Value: !Ref VPC
    Export:
      Name: cfn-workshop-VpcId

  PublicSubnet1:
    Value: !Ref VPCPublicSubnet1
    Export:
      Name: cfn-workshop-PublicSubnet1

  PublicSubnet2:
    Value: !Ref VPCPublicSubnet2
    Export:
      Name: cfn-workshop-PublicSubnet2
```

#### 2. Deploy the VPC Stack

1. Navigate to CloudFormation in the console and click _Create stack With new resources (standard)_.
1. In **Prepare template** select _Template is ready_.
1. In **Template source** select _Upload a template file_.
1. Choose a file `vpc.yaml`.
1. Enter a stack name. For example, cfn-workshop-vpc
1. For the `AvailabilityZones` parameter, select 2 AZs.
1. You can leave the rest of the parameters default.
1. Navigate through the wizard leaving everything default.
1. On the **Review <stack_name>** page, scroll down to the bottom and click on **Create stack**.

### Create IAM Stack

#### 1. Prepare the IAM role template

1. Open `code/70-layered-stack/01-working directory/iam.yaml`.
1. Copy the code below to the _Outputs_ section of the template.

```yaml {hl_lines=[4,5]}
Outputs:
  WebServerInstanceProfile:
    Value: !Ref WebServerInstanceProfile
    Export:
      Name: cfn-workshop-WebServerInstanceProfile
```

#### 2. Deploy the IAM Stack

1. Navigate to CloudFormation in the console and click _Create stack With new resources (standard)_.
1. In **Prepare template** select _Template is ready_.
1. In **Template source** select _Upload a template file_.
1. Choose a file `iam.yaml`.
1. Enter a stack name. For example, cfn-workshop-iam
1. You can leave the rest of the parameters default.
1. Navigate through the wizard leaving everything default.
1. Acknowledge IAM capabilities and click on _Create stack_

### Create EC2 Layered Stack

#### 1. Prepare the EC2 template

The concept of the Layered Stack is to use intrinsic functions to import previously exported values instead of using `Parameters`.
Therefore, the first change to make to the `ec2.yaml` is to remove the parameters that will no longer be used; `SubnetId`, `VpcId`, and `WebServerInstanceProfile`.

Update the _Parameters_ section to look as follows::
```yaml
Parameters:
  EnvironmentType:
    Description: 'Specify the Environment type of the stack.'
    Type: String
    Default: Test
    AllowedValues:
      - Dev
      - Test
      - Prod
    ConstraintDescription: 'Specify either Dev, Test or Prod.'

  AmiID:
    Type: AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>
    Description: 'The ID of the AMI.'
    Default: /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2
```

Next, we need to update the `Fn::Ref` in the template to import the exported values from the vpc and iam stacks created earlier.
We perform this import by using the [Fn::ImportValue](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-importvalue.html) intrinsic function.

Update `WebServerInstance` resource in the _Resources_ section of the `ec2.yaml` template.
```yaml {hl_lines=[5,6]}
  WebServerInstance:
    Type: AWS::EC2::Instance
    {...}
    Properties:
      SubnetId: !ImportValue cfn-workshop-PublicSubnet1
      IamInstanceProfile: !ImportValue cfn-workshop-WebServerInstanceProfile
      ImageId: !Ref AmiID
      InstanceType: !FindInMap [EnvironmentToInstanceType, !Ref EnvironmentType, InstanceType]
    {...}
```

Finally, update the security group resource in a similar way.
Update `WebServerSecurityGroup` resource in the _Resources_ section of the `ec2.yaml` template.
```yaml {hl_lines=[10]}
  WebServerSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: 'Enable HTTP access via port 80'
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
      VpcId: !ImportValue cfn-workshop-VpcId
```

#### 2. Deploy the EC2 Stack

1. Navigate to CloudFormation in the console and click _Create stack With new resources (standard)_.
1. In **Prepare template** select _Template is ready_.
1. In **Template source** select _Upload a template file_.
1. Choose a file `ec2.yaml`.
1. Enter a stack name. For example, cfn-workshop-ec2
1. You can leave the rest of the parameters default.
1. Navigate through the wizard leaving everything default.
1. On the **Review <stack_name>** page, scroll down to the bottom and click on **Create stack**.

## Conclusion

Layered stacks allow you to create resources that can be used again and again in multiple stacks. All the stack needs to know is the Export name used. They allow the separation of roles and responsibilities. For example, a network team could create and supply an approved VPC design as a template. You deploy it as a stack and then just reference the Exports as needed. Similarly, a security team could do the same for IAM roles or EC2 security groups.

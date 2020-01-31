---
title: 'Lab 03: Intrinsic Functions'
date: 2019-11-01T13:36:34Z
weight: 200
---

### Overview

This lab shows how to use **[Intrinsic Functions](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference.html)** in your template. 

Intrinsic functions are built-in functions that help you manage your stacks. Without them, you will be limited to very basic templates, similar to the S3 template you have seen in a **[Lab01](/30-workshop-part-01/10-cloudformation-fundamentals/200-lab-01-stack)**.

### Topics Covered

In this Lab, you will:

+ Use the **[Fn::Ref](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-ref.html)** function to dynamically assign parameter values to a resource property (we have already used this in the last lab!).
+ Tag an instance with **[Fn::Join](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-join.html)** function.
+ Add a tag to the instance using **[Fn::Sub](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-sub.html)** function.

### Start Lab

1. Go to the `code/20-cloudformation-features/` directory.
1. Open the `03-lab03-IntrinsicFunctions.yaml` file.
1. Copy the code as you go through the topics below.

{{% notice note %}} 
Intrinsic functions can only be used in certain parts of a template. You can use intrinsic functions in 
**resource properties, outputs, metadata attributes, and update policy attributes**.
{{% /notice %}}

#### Fn::Ref

In the last lab, we had a parameter declared call `InstanceType`. Then in the InstanceType property on the resource instead of having a string value we had `!Ref InstanceType`. This was making using of the **Fn::Ref** intrinsic function. It passes the value of the parameter to the resource property at runtime.
Also in the last lab you "hardcoded" an AMI ID directly into the EC2 Resource property. You will now amend this to make your template more flexible. Let's convert `ImageId` to a variable and pass it to the resource property at the runtime.

1. First, create a new parameter called `AmiID` and put it in the `Parameters` section of your template. 

    ```yaml
      AmiID:
        Type: AWS::EC2::Image::Id
        Description: 'The ID of the AMI.'
    ```

{{% notice note %}} 
Notice we are declaring the type of the parameter as `AWS::EC2::Image::Id` so that CloudFormation knows to expect a specific type of value and not just a string.
{{% /notice %}}

1. Use the intrinsic function `Ref` to pass the `AmiID` parameter to the EC2 resource property.

    ```yaml
    Resources:
      WebServerInstance:
        Type: AWS::EC2::Instance
        Properties:
          ImageId: !Ref AmiID
          InstanceType: !Ref InstanceType
    ```

#### Fn::Join

The intrinsic function **Fn::Join** appends a set of values into a single value, separated by the specified delimiter. If a delimiter is an empty string the set of values are concatenated with no delimiter.

To help you manage your AWS resources, you can optionally assign metadata to each resource in the form of **tags**. Each tag is a simple label consisting of a customer-defined key and an optional value that can help you to categorize resources by purpose, owner, environment, or other criteria. Let's use the intrinsic function **Fn::Join** to name your instance.

1. Add property `Tags` to the `Properties` section. 
1. Reference `InstanceType` parameter and add a word _webserver_, delimited with a dash `-` to the tags property.

    ```yaml
    Resources:
      WebServerInstance:
        Type: AWS::EC2::Instance
        Properties:
          ImageId: !Ref AmiID
          InstanceType: !Ref InstanceType
          Tags:
            - Key: Name
              Value: !Join [ '-', [ !Ref InstanceType, webserver ] ]
    ```

Here the **Fn::Join** function is concatenating the text value of the InstanceType parameter with the static string `webserver` and a `-` between them. For example, the final value of the Name tag might be `t2.micro-webserver`

#### Update EC2 stack

Now it is time to update your stack. Go to the AWS console and update your CloudFormation Stack.

1. Open the **[AWS CloudFormation](https://console.aws.amazon.com/cloudformation)** link in a new tab and log in to your AWS account.
1. Click on the stack name, for example **cfn-workshop-ec2**.
1. In the top right corner click on **Update**.
1. In **Prepare template**, choose **Replace current template**.
1. In **Template source**, choose **Upload a template file**.
1. Click on **Choose file** button and navigate to your workshop directory.
1. Select the file `03-lab03-IntrinsicFunctions.yaml` and click **Next**.
1. For **Type of EC2 Instance** leave the default value in.
1. For **The ID of the AMI.** copy and paste the ImageId you have hardcoded in `01-lab02-Resources.yaml` file and click **Next**.
1. You can leave **Configure stack options** default, click **Next**.
1. On the **Review <stack_name>** page, scroll down to the bottom and click on **Update stack**.
1. You can click the **refresh** button a few times until you see in the status **UPDATE_COMPLETE**.

{{%expand "Want to see how to update the stack?" %}}
![update-gif](../update-1.gif)
{{% /expand %}}

**To see the result of the stack update:**

1. Open **[AWS EC2 console](https://console.aws.amazon.com/ec2)** link in a new tab of your browser.
1. In the left-hand pane, click on **Instances**.
1. Select the instance with a name **t2.micro-webserver**
1. Go to the **Tags** tab, you should see there a key `Name` with a value `t2.micro-webserver`.

    ![tags-png](../tags.png)

### Challenge
Create another tag named `Name2` and use intrinsic function **Fn::Sub** instead of **Fn::Join** to achieve the same value. 

The syntax for the short form is `!Sub`

{{%expand "Need a hint?" %}}
Check out the AWS Documentation for **[Fn::Sub](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-sub.html)** function.
{{% /expand %}}

{{%expand "Want to see the solution?" %}}

1. Add the `Name2` tag to your template.

    ```yaml
    Resources:
      WebServerInstance:
        Type: AWS::EC2::Instance
        Properties:
          ImageId: !Ref AmiID
          InstanceType: !Ref InstanceType
          Tags:
            - Key: Name
              Value: !Join [ '-', [ !Ref InstanceType, webserver ] ]
            - Key: Name2
              Value: !Sub ${InstanceType}-webserver
    ```

1. Go to the AWS console and update your CloudFormation Stack.
1. In the EC2 console, verify that `Name2` tag has been created.

{{% /expand %}}

---
### Conclusion
Congratulations! You now have successfully used intrinsic functions in your template.

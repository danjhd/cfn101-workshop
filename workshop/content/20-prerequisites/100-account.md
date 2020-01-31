---
title: 'Create an AWS Account'
date: 2019-10-18T12:55:54+01:00
weight: 100
---

## Create an AWS Account for Experimentation

To run this workshop, you'll need access to an AWS account. If you already have an account and your system is already configured with a set of credentials for an administrator user, you can [move to the next step](../200-awscli).

{{% notice warning %}}
If you are using an existing account, either personal or a company account, make sure you understand the implications and policy of provisioning resources into this account.
{{% /notice %}}

If you don't have an AWS account, you can [create a free account here](https://portal.aws.amazon.com/billing/signup).

## Administrator User

{{% notice warning %}}
In this workshop, we will use credentials that have full administrator access to AWS. Having an IAM User configured in this way goes against AWS best practices for [Least Privilege](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html#grant-least-privilege), so please ensure you delete this IAM user at the end of this workshop to reduce the risk as much as possible.
{{% /notice %}}

1. Sign in to your AWS account
1. Go to the AWS IAM console and [create a new user](https://console.aws.amazon.com/iam/home?#/users$new).
1. Type a name for your user (e.g. `cfn-workshop`) and choose both, **Programmatic access** and **AWS Management Console Access**.

    ![new-user-1-png](../new-user-1.png)

1. Click **Next: Permissions** to continue to the next step.
1. Click **Attach existing policies directly** and choose **AdministratorAccess**.

    ![new-user-2-png](../new-user-2.png)

1. Click **Next: Tags**
1. Click **Next: Review**
1. Click **Create User**
1. In the next screen, you'll see your **Access key ID** and you will have the option to click **Show** to show the **Secret access key**.

    ![new-user-3-png](../new-user-3.png)

{{% notice info %}}
Important: Keep this browser window open for the next step or take a note of the access key ID and secret access key.
{{% /notice %}}
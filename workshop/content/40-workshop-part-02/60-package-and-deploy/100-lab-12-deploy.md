---
title: "Lab 12: Deployment using the CLI"
date: 2019-11-25T14:55:21Z
weight: 100
---

## Overview

So far, you have used the console to deploy CloudFormation templates. 
As you have probably noticed, uploading files to S3 manually so you can deploy nested stacks through the console is tedious.

It is also possible to deploy CloudFormation templates using the AWS CLI and AWS SDKs. In this section, you will learn how to use the AWS CLI to work with CloudFormation templates.
This section will cover three key commands, used to package, validate and deploy CloudFormation templates with the AWS CLI.

## Packaging a template

Cloudformation components often reference external files in S3. An example of this the `AWS::CloudFormation::Stack` resource. The Component requires the target template to be in S3. What if the external files are on your local machine? In [Lab 10](../../40-nested-stack/100-lab-10-nested-stacks), you uploaded these templates manually to S3 before deploying.

[`aws cloudformation package`](https://docs.aws.amazon.com/cli/latest/reference/cloudformation/package.html) is a useful command that solves this problem. When given a template that references local resources, it will upload the resources to a specified S3 bucket. An updated template is output. The local file references in the template are updated to reference the uploaded assets in S3.

When you package a template, you are required to specify an S3 Bucket to package the contents to.

Let's try it out. Here is an example of using the [`aws cloudformation package`](https://docs.aws.amazon.com/cli/latest/reference/cloudformation/package.html) command. Make sure you insert the name of your S3 bucket in place of `{{example-bucket-name}}`

```bash
aws cloudformation package --template-file code/60-package-and-deploy/01-lab12-deploy.yaml --s3-bucket {{example-bucket-name}} --output-template-file code/60-package-and-deploy/01-lab12-deploy-packaged.yaml --region eu-west-2
```

{{% notice warning %}}
Make sure to change the `--region` flag to the region you are using.
{{% /notice %}}

By default, the updated template is written to the standard output. Use the option `--output-template-file` to specify a path to write the updated CloudFormation template.

Take a look in the S3 bucket. You will find 3 new files. These are the Nested stack templates just like we manually uploaded in [Lab 10](../../40-nested-stack/100-lab-10-nested-stacks) but in this case, the upload was handled completely for you.

Using `aws cloudformation package` you can easily prepare your nested stack for deployment.

## Validating a template

Sometimes a CloudFormation template deployment will fail due to syntax errors in the template.
[`aws cloudformation validate-template`](https://docs.aws.amazon.com/cli/latest/reference/cloudformation/validate-template.html) checks a CloudFormation template to ensure it is valid JSON or YAML. This is useful to speed up development time. 

Let's validate a template.

```bash
aws cloudformation validate-template --template-body file://code/60-package-and-deploy/02-lab12-bad-template.yaml
```

Notice what happens!  

Try to fix the errors, then validate the template again.

If you want to see the fixed version you can find it here `code/60-package-and-deploy/02-lab12-bad-template-Solution.yaml`

## Deploying a template using the CLI

The [`aws cloudformation deploy`](https://docs.aws.amazon.com/cli/latest/reference/cloudformation/deploy/index.html) command is used to deploy CloudFormation templates using the CLI.
When used, it requires a template to be passed to it. This can be either a file in S3 or on the local machine.

You can use the `--parameter-overrides` option to specify parameters in the template. This can be a string containing `'key=value'` pairs or a via a [supplied JSON file](https://docs.aws.amazon.com/cli/latest/userguide/cli-usage-parameters.html#cli-usage-parameters-json).

Let's deploy a CloudFormation template using the CLI.

```bash
aws cloudformation deploy --template-file code/60-package-and-deploy/01-lab12-deploy-packaged.yaml --stack-name cfn-workshop-deploy --parameter-overrides "EnvType=Prod" --capabilities CAPABILITY_IAM --region eu-west-2
```

{{% notice warning %}}
Make sure to change the `--region` flag to the region you are using.
{{% /notice %}}

### Capabilities

You may recall when using the console, you are required you to acknowledge that deploying this template may create IAM resources that can affect permissions in your account. This is to ensure you don't accidentally change the permissions unintentionally.

When using the CLI, you are also required to acknowledge this stack might create resources that can affect IAM permissions. This is done using the `--capabilities` flag, as demonstrated in the previous example. Read more about the possible capabilities in the [`aws cloudformation deploy` documentation](https://docs.aws.amazon.com/cli/latest/reference/cloudformation/deploy/index.html)

---

### Clean up

Follow these steps to clean up created resources:

1. In in the **[CloudFormation console](https://console.aws.amazon.com/cloudformation)**, select one of the stacks you have created in this lab. For example `cfn-workshop-nested-stack`.
1. In the top right corner, click on **Delete**.
1. In the pop-up window click on **Delete stack**.
1. You can click the **refresh** button a few times until you see in the status **DELETE_COMPLETE**.
1. Repeat for all the other stacks created in the workshop.

---

## Conclusion

Using the CLI is a powerful way to deploy CloudFormation templates. The `package` command simplifies the deployment of templates that use features such as nested stacks or refer to other local assets. The `validate` command can speed up the development of templates by catching errors more quickly. The `deploy` command allows you to deploy CloudFormation templates in and automated and unattended way, such as part of a CI/CD pipeline.

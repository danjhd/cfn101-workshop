---
title: 'Default VPC'
date: 2020-01-30T11:33:42+01:00
weight: 600
---

Part 01 of this workshop requires that a default VPC is available in the region being used.

We will use the AWS CLI to list all existing VPCs in the region and let us know if a default VPC is present or not. 

Copy the code below to your terminal. Make sure to change the `--region` flag to use a region that you are going to be deploying your CloudFormation to.

```bash
aws ec2 describe-vpcs --filters Name=isDefault,Values=true --query "Vpcs[].VpcId" --region eu-west-2
```

{{%expand "Need help?" %}}
![default-vpc-gif](../default-vpc.gif)
{{% /expand %}}

If the response shows as `[]` then this means there is no default VPC in this region. If the response is not empty you can [move to the next step](../../30-workshop-part-01).

We will also use the AWS CLI to create a new default VPC if one does not exist. 

Copy the code below to your terminal. Make sure to change the `--region` flag to use a region that you are going to be deploying your CloudFormation to.

```bash
aws ec2 create-default-vpc --region eu-west-2
```
The result will be a new default VPC being created and the response in the terminal will look like the sample below.

```bash
{
    "Vpc": {
        "CidrBlock": "172.31.0.0/16",
        "DhcpOptionsId": "dopt-c1422ea9",
        "State": "pending",
        "VpcId": "vpc-088b5ae6628fbf3ac",
        "OwnerId": "123456789012",
        "InstanceTenancy": "default",
        "Ipv6CidrBlockAssociationSet": [],
        "CidrBlockAssociationSet": [
            {
                "AssociationId": "vpc-cidr-assoc-0ab2ffabcbe0548bc",
                "CidrBlock": "172.31.0.0/16",
                "CidrBlockState": {
                    "State": "associated"
                }
            }
        ],
        "IsDefault": true,
        "Tags": []
    }
}
```

{{%notice note %}}
If you wish to delete the default VPC again at the end of this workshop you should make a note of the VpcId above so that you can be sure to know which one to delete later.
{{% /notice %}}

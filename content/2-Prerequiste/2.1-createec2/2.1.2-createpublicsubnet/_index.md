---
title : "Enable AWS Config"
date : "`r Sys.Date()`"
weight : 2
chapter : false
pre : " <b> 2.1.2 </b> "
---

#### Enable AWS Config

In the AWS Management Console, open AWS Config.

Click Settings (or Get started if this is your first time enabling the service).

Under Recording method, select All resource types with customizable overrides.

Under Recording frequency, choose Continuous recording.

(Optional) Click Remove under All globally recorded IAM resource types if you want AWS Config to also record IAM users, groups, roles, and policies.

In Data governance → IAM role for AWS Config:

If you don’t have a role yet, select Create AWS Config service-linked role.

If you already have the role AWSServiceRoleForConfig, select Use an existing AWS Config service-linked role.

In Delivery channel → Amazon S3 bucket, choose Choose a bucket from your account and select the bucket you created in 2.1.1 (compliance-audit-logs-<yourname>).

Prefix: leave blank so AWS will automatically create the default path.

Leave Amazon SNS topic empty unless you want notifications via SNS.


![VPC](/images/2.prerequisite/016-creatertb.png)


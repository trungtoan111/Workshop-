---
title : "Create AWS Config Rules"
date : "`r Sys.Date()`"
weight : 3
chapter : false
pre : " <b> 2.1.3 </b> "
---

#### Create AWS Config Rules
In this step, you will add AWS Managed Rules to evaluate your AWS resources against compliance requirements. These rules are pre-built by AWS and allow you to check for common security, logging, and encryption best practices without writing custom code.

Steps:

In the AWS Management Console, go to AWS Config → Rules.

Click Add rule.

Select AWS managed rules from the rule type options.

In the search bar, type each of the following rule names exactly and add them to your environment:

Security & Network

restricted-ssh — Checks whether security groups disallow unrestricted incoming SSH traffic (port 22).

vpc-flow-logs-enabled — Ensures VPC Flow Logs are enabled for all VPCs.

eip-attached — Checks whether all Elastic IPs are attached to an instance or network interface.

Logging & Monitoring

cloud-trail-enabled — Ensures at least one CloudTrail is enabled in your account.

cloud-trail-log-file-validation-enabled — Ensures log file validation is enabled on all CloudTrail trails.

multi-region-cloudtrail-enabled — Ensures CloudTrail is enabled in all regions.

Encryption & Data Protection

s3-bucket-server-side-encryption-enabled — Ensures all S3 buckets have encryption at rest enabled.

ec2-ebs-encryption-by-default — Ensures EBS volume encryption is enabled by default.

rds-storage-encrypted — Ensures all RDS instances are encrypted.

Leave parameters as default unless your compliance requirements specify otherwise.

Click Add rule to save them.

Return to the Rules list to verify they are in an Active state.

![VPC](/images/2.prerequisite/018-createsubnet.png)





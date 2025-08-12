---
title : "Review and Confirm AWS Config Setup"
date : "`r Sys.Date()`"
weight : 4
chapter : false
pre : " <b> 2.1.4 </b> "
---

In the AWS Config console, after adding all required rules, click Next.

Review your configuration in the Review page:

Recording method → Record all resource types with customizable overrides.

Default recording frequency → Continuous.

S3 bucket → compliance-audit-logs-yourname.

AWS Config rules → Ensure the list contains all compliance rules you added (e.g., restricted-ssh, vpc-flow-logs-enabled, eip-attached, cloud-trail-log-file-validation-enabled, multi-region-cloudtrail-enabled, s3-bucket-server-side-encryption-enabled, ec2-ebs-encryption-by-default, rds-storage-encrypted).

If all details are correct, click Confirm to save the configuration.

Wait for AWS Config to evaluate resources against your rules.

Go to the Rules page to verify that the rules are active and start showing compliance results.
![SG](/images/2.prerequisite/026-createsg.png)





---
title : "Create an S3 Bucket for Audit Logs"
date : "`r Sys.Date()`"
weight : 1
chapter : false
pre : " <b> 2.1.1 </b> "
---


#### Create an S3 Bucket for Audit Logs
In the AWS Management Console, navigate to Amazon S3 → Create bucket.

Enter a globally unique bucket name (e.g., compliance-audit-logs-<yourname>).

Select the AWS Region you are using for this lab.

Enable Bucket Versioning.

Enable Server-Side Encryption (SSE-KMS or SSE-S3).

Block all public access.

Click Create bucket.

![VPC](/images/2.prerequisite/002-createvpc.png)


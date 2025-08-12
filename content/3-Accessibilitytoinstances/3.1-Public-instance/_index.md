---
title : "Connect AWS Config to Network Resources"
date : "`r Sys.Date()`"
weight : 1
chapter : false
pre : " <b> 3.1. </b> "
---
Goal: Make sure AWS Config is actively recording your core network resources and writing snapshots to S3.

Steps

Open AWS Config → Settings.

Under Recording method, select All resource types with customizable overrides.

(Optional) Remove the exclusion for All globally recorded IAM resource types if you want IAM to be recorded as well.

Under Recording frequency, keep Continuous recording.

In Delivery channel, confirm the S3 bucket from Step 2.1.1 (e.g., compliance-audit-logs-<yourname>) is selected.

Click Save/Confirm. If the recorder is off, go back to Settings → Recorder and click Start recording.

Verify the following resource families are being recorded (either via “All resource types” or by checking the recorded inventory):

VPC

Security Groups

Network ACLs (NACLs)

Elastic Load Balancers (ALB/NLB)

S3 Buckets

Validate snapshots/logs:

In AWS Config → Dashboard, confirm Configuration Items Recorded is increasing.

In S3, check that objects appear under AWSLogs/<account-id>/Config/<region>/....

Expected result: AWS Config is actively recording your network resources and writing configuration snapshots to the S3 bucket.

![Connect](/images/3.connect/004-connect.png)


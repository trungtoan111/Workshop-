---
title : "Manage and Analyze Audit Logs"
date : "`r Sys.Date()`"
weight : 4
chapter : false
pre : " <b> 4. </b> "
---


**Goal**  
Capture and monitor **AWS Systems Manager Session Manager** activity for private EC2 instances. You will enable **logging to S3/CloudWatch Logs**, then query activity using **CloudWatch Logs Insights** and **Athena (CloudTrail)** for audit and forensics.

> **Prerequisites**
> - Logs bucket from Section 2 (e.g., `network-compliance-logs-<account>-<region>`).  
> - VPC Interface Endpoints created: `ssm`, `ssmmessages`, `ec2messages`.  
> - Instances have **SSM Agent** and the IAM role **AmazonSSMManagedInstanceCore**.  
> - Endpoint Security Group allows **TCP 443** from your private subnets/VPC CIDR.  
> - VPC **DNS resolution/hostnames** enabled; **Private DNS** enabled on endpoints.

---

## Enable Session Logging (Preferences)

**Console**
1. **Systems Manager → Session Manager → Preferences → Edit**  
2. **S3**: set `s3://network-compliance-logs-<account>-<region>/ssm-logs/`  
   - *(Optional)* choose a **KMS key** for encryption  
3. **CloudWatch Logs**: enable and choose a log group (e.g., `/ssm/sessions`)  
4. *(Recommended)* set **Idle timeout** (e.g., 20m) and **Max session duration** (e.g., 60m)  
5. **Save**

📸 Upload later:
- `/images/4-4-ssm-logging-prefs.png` *(Preferences with S3 + CW Logs)*  
- `/images/4-4-ssm-logs-s3.png` *(S3 prefix `/ssm-logs/` populated)*  
- `/images/4-4-ssm-logs-cw.png` *(CloudWatch log group `/ssm/sessions`)*

> **If S3 uploads fail with AccessDenied**, add a bucket policy to allow `ssm.amazonaws.com` to `PutObject` under `ssm-logs/*` (and set `x-amz-acl: bucket-owner-full-control`):

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "AllowSSMToWriteSessionLogs",
    "Effect": "Allow",
    "Principal": { "Service": "ssm.amazonaws.com" },
    "Action": "s3:PutObject",
    "Resource": "arn:aws:s3:::NETWORK_COMPLIANCE_BUCKET/ssm-logs/*",
    "Condition": { "StringEquals": { "s3:x-amz-acl": "bucket-owner-full-control" } }
  }]
}
```
## Monitor with CloudWatch Logs Insights (live/search)
Open **CloudWatch → Logs Insights**, select the group /ssm/sessions, then run queries:

## A) List recent sessions

sql

fields @timestamp, @logStream
| sort @timestamp desc
| limit 50

## B) Search sensitive commands (Linux)

sql

fields @timestamp, @message
| filter @message like /sudo|passwd|useradd|iptables|ssh|fail|denied|permission/i
| sort @timestamp desc
| limit 200

## C) Windows PowerShell risky patterns

sql
fields @timestamp, @message
| filter @message like /Add-LocalUser|Set-LocalUser|New-NetFirewallRule|RDP|Enable-PSRemoting/i
| sort @timestamp desc
| limit 200

## Audit with CloudTrail (who started/ended sessions) via Athena

All StartSession/TerminateSession API calls are in CloudTrail. Use Athena (CloudTrail table from 4.3):

## A) Session opens & closes

sql

SELECT
  from_iso8601_timestamp(eventtime) AS event_time,
  useridentity.arn                  AS actor,
  eventname,
  requestparameters.target          AS instance_id,
  requestparameters.documentName    AS document,
  sourceipaddress                   AS source_ip
FROM compliance_logs.<your_cloudtrail_table>
WHERE eventsource = 'ssm.amazonaws.com'
  AND eventname IN ('StartSession','TerminateSession')
  AND from_iso8601_timestamp(eventtime) >= date_add('day', -7, current_timestamp)
ORDER BY event_time DESC;

## B) Port-forwarding sessions (RDP/SSH tunneling)

sql

SELECT
  from_iso8601_timestamp(eventtime) AS event_time,
  useridentity.arn                  AS actor,
  requestparameters.documentName    AS document,
  requestparameters.parameters      AS params
FROM compliance_logs.<your_cloudtrail_table>
WHERE eventsource = 'ssm.amazonaws.com'
  AND eventname   = 'StartSession'
  AND requestparameters.documentName = 'AWS-StartPortForwardingSession'
ORDER BY event_time DESC;

## CLI quick checks
**List active sessions**

```bash

REGION=ap-southeast-1
aws ssm describe-sessions --region "$REGION" --state Active --max-results 50
```

**List recent sessions (history)**

```bash

aws ssm describe-sessions --region "$REGION" --state History --max-results 50
```
## Get a specific session’s details

```bash

SESSION_ID=<your-session-id>
aws ssm get-session --region "$REGION" --session-id "$SESSION_ID"
```

---
title : "Enable AWS Config"
date : "`r Sys.Date()`"
weight : 2
chapter : false
pre : " <b> 2.1.2 </b> "
---

**Goal**  
Enable **AWS Config** to continuously record resource configuration (with a focus on network resources) and deliver snapshots/history to the **S3 audit bucket** from 2.1.1.

> **Prerequisites**
> - The S3 logs bucket exists (e.g., `network-compliance-logs-<account>-<region>`) with versioning + encryption.
> - You have permissions for **Config**, **S3**, and **IAM** (to create the service-linked role if needed).

---

## Console

1) Open **AWS Config** → **Get started** (or **Settings** if already enabled).  
2) **Resource recording**  
   - **Record all supported resources** *(recommended)*, or select key network types (VPC, Subnet, RouteTable, Internet/NAT GW, VPCEndpoint, SecurityGroup, NACL, ENI, ELB/TargetGroup, EIP, S3).  
   - *(Optional)* **Include global resources** (IAM, etc.) – keep **off** for a network-focused lab.
3) **Delivery channel**  
   - **S3 bucket**: `network-compliance-logs-<account>-<region>`  
   - **S3 prefix**: `config-logs/`  
   - *(Optional)* **KMS key** for SSE-KMS.
4) **Permissions**  
   - Let AWS create/use the **service-linked role** `AWSServiceRoleForConfig`.
5) **Start recording** and confirm status shows **Recording**.

📸 Upload later:
- `/images/2-1-2-config-get-started.png` *(Get started page)*
- `/images/2-1-2-config-settings.png` *(Recorder: Record all supported resources)*
- `/images/2-1-2-config-delivery.png` *(S3 bucket + prefix = config-logs/)*
- `/images/2-1-2-config-status.png` *(Recorder status = Recording)*

> After a few minutes, objects should appear under  
> `s3://network-compliance-logs-<account>-<region>/config-logs/AWSLogs/<ACCOUNT_ID>/Config/<region>/...`

---

## CLI (optional)

```bash
REGION="ap-southeast-1"
ACCOUNT_ID="$(aws sts get-caller-identity --query Account --output text)"
BUCKET="network-compliance-logs-$ACCOUNT_ID-$REGION"

# 0) Ensure the AWS Config service-linked role exists (safe if already created)
aws iam create-service-linked-role --aws-service-name config.amazonaws.com >/dev/null 2>&1 || true

ROLE_ARN="arn:aws:iam::$ACCOUNT_ID:role/aws-service-role/config.amazonaws.com/AWSServiceRoleForConfig"

# 1) Create/Update the configuration recorder
aws configservice put-configuration-recorder --region "$REGION" --configuration-recorder "{
  \"name\": \"default\",
  \"roleARN\": \"$ROLE_ARN\",
  \"recordingGroup\": {
    \"allSupported\": true,
    \"includeGlobalResourceTypes\": false
  }
}"

# 2) Set the delivery channel to the logs bucket
aws configservice put-delivery-channel --region "$REGION" --delivery-channel "{
  \"name\": \"default\",
  \"s3BucketName\": \"$BUCKET\",
  \"s3KeyPrefix\": \"config-logs\"
}"

# 3) Start recording
aws configservice start-configuration-recorder --region "$REGION" --configuration-recorder-name default

# 4) Quick status check
aws configservice describe-configuration-recorder-status --region "$REGION"
Make sure your bucket policy allows config.amazonaws.com to PutObject into config-logs/AWSLogs/<ACCOUNT_ID>/* (see 2.1.1).
```

**Validate**
**S3** shows new keys under config-logs/AWSLogs/<ACCOUNT_ID>/Config/<region>/...

**AWS Config → Resource inventory** lists VPCs, Subnets, Security Groups, etc.

**AWS Config → Settings shows Recording** = On and the correct S3 destination.

**Good practices**
Prefer Record all supported resources to catch dependencies beyond networking.

Keep logs private, versioned, and encrypted; apply lifecycle to archive after 90–180 days.

In multi-account/Region environments, plan a Config Aggregator in a home Region (set up later) for centralized inventory/compliance.


---
title : "Manage and Analyze Audit Logs"
date : "`r Sys.Date()`"
weight : 4
chapter : false
pre : " <b> 4. </b> "
---


**Goal**  
Centralize and analyze **audit & compliance logs** from **CloudTrail** and **AWS Config** in **Amazon S3**, query them with **Athena**, and prepare datasets for **QuickSight** dashboards.

**By the end you will**
- Store logs in a secure, versioned **S3** bucket (from Section 2).  
- Enable **CloudTrail (multi-Region + validation)** to capture all API activity.  
- Catalog log data with **AWS Glue** and query with **Amazon Athena**.  
- (Optional) Prepare **QuickSight** datasets and enable **Session Manager** session logging for audit evidence.

### Architecture
![Logs architecture – S3 + CloudTrail + Config + Athena]( /images/4-arch-logs.png )

---

## 4.1 Store Audit Logs in Amazon S3

Use the bucket you created earlier (e.g., `network-compliance-logs-<account>-<region>`) and organize prefixes:

- `cloudtrail-logs/` – CloudTrail management/data events  
- `config-logs/` – AWS Config snapshots & configuration history  
- `securityhub-findings/` *(optional)* – if you export findings via EventBridge→Firehose/Lambda  
- `ssm-logs/` *(optional)* – Session Manager session logs

**Bucket settings (recap)**
- **Block Public Access:** ON (all 4)  
- **Versioning:** Enabled  
- **Default Encryption:** SSE-S3 or SSE-KMS (enable *Bucket Key* if KMS)

📸 Upload later:
- `/images/4-1-s3-layout.png` *(Prefixes in the logs bucket)*

---

## 4.2 Enable AWS CloudTrail for Full Logging

**Console**
1. Open **CloudTrail → Trails → Create trail**  
2. **Name:** `network-compliance-trail`  
3. **Storage location:** your logs bucket → prefix `cloudtrail-logs/`  
4. **Apply trail to all Regions:** **Yes** (multi-Region)  
5. **Log file validation:** **Enabled**  
6. **Event type:** Management events = **Read/Write**  
7. *(Optional)* **Data events** – start with S3 buckets that contain sensitive data and **Lambda** functions handling prod traffic.  
8. **Create trail**

📸 Upload later:
- `/images/4-2-cloudtrail-create.png` *(Create trail – settings)*
- `/images/4-2-event-selectors.png` *(Data events selectors)*

**CLI**
```bash
REGION=ap-southeast-1
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
BUCKET="network-compliance-logs-$ACCOUNT_ID-$REGION"
TRAIL="network-compliance-trail"

# Create a multi-Region trail with log file validation
aws cloudtrail create-trail \
  --name "$TRAIL" \
  --s3-bucket-name "$BUCKET" \
  --s3-key-prefix "cloudtrail-logs" \
  --is-multi-region-trail \
  --include-global-service-events \
  --enable-log-file-validation

# Start logging
aws cloudtrail start-logging --name "$TRAIL"

# Quick check
aws cloudtrail get-trail-status --name "$TRAIL"

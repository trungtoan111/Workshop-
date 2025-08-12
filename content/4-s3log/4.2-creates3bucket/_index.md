---
title : "Enable AWS CloudTrail for Full Logging"
date : "`r Sys.Date()`"
weight : 2
chapter : false
pre : " <b> 4.2 </b> "
---


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
```

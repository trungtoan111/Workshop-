---
title   : "Create IAM Roles and Policies"
date    : "`r Sys.Date()`"
weight  : 1
chapter : false
pre     : " <b> 2.1 </b> "
---

*Goal**  
Baseline **AWS Config** so network resources are continuously **recorded**, evaluated against **managed rules / conformance packs**, and (optionally) **auto-remediated**. This is the foundation for continuous compliance and auditability.

### Why this matters
- **Always-on evidence** for audits (snapshots/history in S3)
- **Fast drift detection** on VPC/SG/NACL/ELB/S3
- **One-click fixes** via native remediation or EventBridge → Lambda (later in Section 5)

### Architecture
![AWS Config baseline – recorder, rules, S3, Security Hub](/images/2-1-arch-config.png)

---

## What you will verify in 2.1
- **Recorder & delivery**: Config is **Recording = On** and writing to **S3 `config-logs/`**
- **Rules**: high-value managed rules are **Evaluating → Compliant/Noncompliant**
- **CloudTrail**: enabled (set up next) to capture API evidence
- **Security Hub**: will consume Config-backed controls later in Section 3
- **IAM roles**: service-linked role for Config, plus roles for remediation/reporting

---

## Quick health checklist (Console)
- **AWS Config → Settings**:  
  - Recorder **On**, *All supported resources* (or key network types)  
  - Delivery channel → S3 bucket `network-compliance-logs-<account>-<region>` + prefix `config-logs/`
- **AWS Config → Rules**: rules show recent **Last successful evaluation** timestamps
- **Amazon S3**: objects appear under `config-logs/AWSLogs/<ACCOUNT_ID>/Config/<region>/...`

📸 Upload later:
- `/images/2-1-config-settings.png`  
- `/images/2-1-rules-status.png`  
- `/images/2-1-s3-config-keys.png`

---

## One-minute CLI sanity check
```bash
REGION=ap-southeast-1
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
BUCKET="network-compliance-logs-$ACCOUNT_ID-$REGION"

echo "== Recorder & Delivery =="
aws configservice describe-configuration-recorder-status --region "$REGION"
aws configservice describe-delivery-channel-status --region "$REGION"
echo "== S3 evidence =="
aws s3 ls "s3://$BUCKET/config-logs/AWSLogs/$ACCOUNT_ID/Config/$REGION/" --recursive | head -n 10
echo "== Rule compliance summary =="
aws configservice get-compliance-summary-by-config-rule --region "$REGION"

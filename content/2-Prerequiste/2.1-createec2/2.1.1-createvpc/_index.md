---
title : "Create an S3 Bucket for Audit Logs"
date : "`r Sys.Date()`"
weight : 1
chapter : false
pre : " <b> 2.1.1 </b> "
---


**Goal**  
Create a **secure, versioned S3 bucket** to store **CloudTrail logs**, **AWS Config snapshots/evaluations**, and optional exports (Security Hub, Session Manager).

> **Prerequisites:** You have permissions for S3 and KMS (if using SSE-KMS).

---

## Console

1. **S3 → Create bucket**
   - **Bucket name:** `network-compliance-logs-<account>-<region>` *(must be globally unique)*
   - **AWS Region:** your lab Region  
   - **Object Ownership:** **Bucket owner enforced** (ACLs disabled)  
   - **Block Public Access:** **ON (all 4)**  
   - **Versioning:** **Enable**  
   - **Default encryption:** **SSE-S3 (AES-256)** or **SSE-KMS** (recommended for regulated data)
   - *(If SSE-KMS)* enable **Bucket Key** to reduce KMS costs
   - **Create bucket**

📸 Upload later:
- `/images/2-1-1-s3-create-bucket.png` *(Create bucket screen)*
- `/images/2-1-1-s3-block-public.png`  *(Block Public Access = ON)*
- `/images/2-1-1-s3-versioning.png`    *(Versioning enabled)*
- `/images/2-1-1-s3-encryption.png`    *(Default encryption)*

**Recommended prefixes (folders)**
- `cloudtrail-logs/` – CloudTrail management & data events  
- `config-logs/` – AWS Config snapshots/history/evaluations  
- `securityhub-findings/` *(optional)*  
- `ssm-logs/` *(optional, Session Manager)*

---

## CLI (optional)

```bash
ACCOUNT_ID="$(aws sts get-caller-identity --query Account --output text)"
REGION="ap-southeast-1"
BUCKET="network-compliance-logs-$ACCOUNT_ID-$REGION"

# 1) Create bucket
aws s3api create-bucket \
  --bucket "$BUCKET" \
  --region "$REGION" \
  --create-bucket-configuration LocationConstraint="$REGION"

# 2) Block public access
aws s3api put-public-access-block \
  --bucket "$BUCKET" \
  --public-access-block-configuration '{
    "BlockPublicAcls":true,
    "IgnorePublicAcls":true,
    "BlockPublicPolicy":true,
    "RestrictPublicBuckets":true
  }'

# 3) Enable versioning
aws s3api put-bucket-versioning \
  --bucket "$BUCKET" \
  --versioning-configuration Status=Enabled

# 4) Default encryption (SSE-S3). For SSE-KMS, swap to your CMK ARN.
aws s3api put-bucket-encryption \
  --bucket "$BUCKET" \
  --server-side-encryption-configuration '{
    "Rules":[{"ApplyServerSideEncryptionByDefault":{"SSEAlgorithm":"AES256"}}]
  }'

# 5) Create logical prefixes
for p in cloudtrail-logs/ config-logs/ securityhub-findings/ ssm-logs/; do
  aws s3api put-object --bucket "$BUCKET" --key "$p" >/dev/null
done
```
**Bucket policy for CloudTrail & AWS Config**
Replace NETWORK_COMPLIANCE_BUCKET and ACCOUNT_ID.
If you use SSE-KMS for CloudTrail, also update the KMS key policy to allow CloudTrail.

```json

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AWSCloudTrailAclCheck",
      "Effect": "Allow",
      "Principal": { "Service": "cloudtrail.amazonaws.com" },
      "Action": "s3:GetBucketAcl",
      "Resource": "arn:aws:s3:::NETWORK_COMPLIANCE_BUCKET"
    },
    {
      "Sid": "AWSCloudTrailWrite",
      "Effect": "Allow",
      "Principal": { "Service": "cloudtrail.amazonaws.com" },
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::NETWORK_COMPLIANCE_BUCKET/cloudtrail-logs/AWSLogs/ACCOUNT_ID/*",
      "Condition": { "StringEquals": { "s3:x-amz-acl": "bucket-owner-full-control" } }
    },
    {
      "Sid": "AWSConfigBucketPermissionsCheck",
      "Effect": "Allow",
      "Principal": { "Service": "config.amazonaws.com" },
      "Action": "s3:GetBucketAcl",
      "Resource": "arn:aws:s3:::NETWORK_COMPLIANCE_BUCKET"
    },
    {
      "Sid": "AWSConfigWrite",
      "Effect": "Allow",
      "Principal": { "Service": "config.amazonaws.com" },
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::NETWORK_COMPLIANCE_BUCKET/config-logs/AWSLogs/ACCOUNT_ID/*"
    }
  ]
}
```
(Optional) Session Manager S3 write permission (if you enable SSM session logging later):

```json

{
  "Sid": "AllowSSMToWriteSessionLogs",
  "Effect": "Allow",
  "Principal": { "Service": "ssm.amazonaws.com" },
  "Action": "s3:PutObject",
  "Resource": "arn:aws:s3:::NETWORK_COMPLIANCE_BUCKET/ssm-logs/*",
  "Condition": { "StringEquals": { "s3:x-amz-acl": "bucket-owner-full-control" } }
}
```
**Good practices**
**Lifecycle rules**: transition logs to Glacier/Deep Archive after 90–180 days; retain per policy.

**Least privilege:** restrict who can read the bucket; keep Public Access Block ON.

**Naming:** one bucket per account/Region simplifies IAM and Athena partitioning.

**KMS (recommended):** use CMK with key policy granting CloudTrail/Config (and your log readers) access.

**Validate**
In S3, confirm the bucket exists with Versioning + Encryption enabled.

After enabling CloudTrail/AWS Config (next steps), expect keys under:

cloudtrail-logs/AWSLogs/<ACCOUNT_ID>/...

config-logs/AWSLogs/<ACCOUNT_ID>/...

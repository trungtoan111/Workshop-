---
title   : "Review and Confirm AWS Config Setup"
date    : "`r Sys.Date()`"
weight  : 4
chapter : false
pre     : " <b> 2.1.4 </b> "
---

**Goal**  
Verify that **AWS Config** is recording the right resources, delivering snapshots/history to your **S3 audit bucket**, and that your **rules** are evaluating without errors.

> **Prerequisites:** Completed **2.1.1** (S3 bucket), **2.1.2** (Enable Config), **2.1.3** (Create rules).

---

## Quick health checklist

- **Recorder:** `Recording = On`, **All supported resources** *(or your selected network types)*  
- **Delivery channel:** S3 bucket = `network-compliance-logs-<account>-<region>`, prefix = `config-logs/`  
- **Service-linked role:** `AWSServiceRoleForConfig` exists  
- **S3 evidence:** keys under `config-logs/AWSLogs/<ACCOUNT_ID>/Config/<region>/...`  
- **Rules:** status moves *Evaluating → Compliant/Noncompliant*; no evaluation errors

---

## A) Verify in Console

1. **AWS Config → Settings**  
   - **Configuration recorder:** **On**  
   - **Recorded resources:** *All supported* (or confirm selected network types)  
   - **Delivery channel:** points to your **S3 bucket + `config-logs/` prefix**
2. **AWS Config → Resource inventory**  
   - Filter by **VPC, Subnet, SecurityGroup, NACL, ENI, LoadBalancer, S3** and confirm items appear.
3. **AWS Config → Rules**  
   - Open a few rules (e.g., **RESTRICTED_SSH**, **S3_BUCKET_SERVER_SIDE_ENCRYPTION_ENABLED**) and confirm **Last successful evaluation** timestamps and **Compliance**.
4. **Amazon S3 → your logs bucket**  
   - Browse to `config-logs/AWSLogs/<ACCOUNT_ID>/Config/<region>/` and confirm objects exist.

📸 Upload later:
- `/images/2-1-4-config-settings.png` *(Recorder + Delivery channel)*
- `/images/2-1-4-config-inventory.png` *(Resource inventory shows network resources)*
- `/images/2-1-4-rule-compliance.png` *(Rule page with evaluation results)*
- `/images/2-1-4-s3-config-keys.png` *(S3 keys under config-logs/)*

---

## B) Verify via CLI

```bash
REGION="ap-southeast-1"
ACCOUNT_ID="$(aws sts get-caller-identity --query Account --output text)"
BUCKET="network-compliance-logs-$ACCOUNT_ID-$REGION"

# 1) Recorder status
aws configservice describe-configuration-recorder-status --region "$REGION"

# 2) Delivery channel status (look for lastSuccessful* fields and error codes)
aws configservice describe-delivery-channel-status --region "$REGION"

# 3) List a few delivered objects in S3
aws s3 ls "s3://$BUCKET/config-logs/AWSLogs/$ACCOUNT_ID/Config/$REGION/" --recursive | head -n 20

# 4) Compliance summary by rule
aws configservice get-compliance-summary-by-config-rule --region "$REGION"

# 5) Compliance summary by resource type (focus on network)
aws configservice get-compliance-summary-by-resource-type \
  --region "$REGION" \
  --resource-types \
    AWS::EC2::VPC AWS::EC2::Subnet AWS::EC2::SecurityGroup AWS::EC2::NetworkAcl \
    AWS::EC2::InternetGateway AWS::EC2::NatGateway AWS::EC2::VPCEndpoint \
    AWS::ElasticLoadBalancingV2::LoadBalancer AWS::S3::Bucket

# 6) Drill into a specific rule's evaluations (example)
aws configservice get-compliance-details-by-config-rule \
  --region "$REGION" \
  --config-rule-name "s3-bucket-encryption-required" \
  --limit 50
```

**Tip: To quickly spot problems, query error fields:**

```bash

aws configservice describe-delivery-channel-status \
  --region "$REGION" \
  --query 'DeliveryChannelsStatus[0].configHistoryDeliveryInfo.[lastErrorCode,lastErrorMessage,lastStatusChangeTime]'
```

## C) Common fixes
Missing service-linked role

```bash

aws iam create-service-linked-role --aws-service-name config.amazonaws.com || true
```
**S3 AccessDenied**

Update the bucket policy to allow config.amazonaws.com s3:PutObject into config-logs/AWSLogs/<ACCOUNT_ID>/* (see 2.1.1).

If using SSE-KMS, ensure the KMS key policy permits AWS Config.

**No resources in inventory**

Recorder not started or wrong scope. Set All supported resources (recommended) and Start recording.

**Rules stuck Evaluating / errors**

Some managed rules need parameters (ports, VPCs). Edit the rule and set Input parameters appropriately.

Ensure prerequisite services (e.g., Flow Logs for VPC) are configured where required.

## D) Good practices
Keep All supported resources to catch dependencies beyond networking.

Tag rules with Compliance=Network to slice reporting later.

In multi-account/Region setups, plan a Config Aggregator in a home Region for centralized views (optional).


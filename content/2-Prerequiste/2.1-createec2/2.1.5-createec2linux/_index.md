---
title : "Verify Compliance Status"
date : "`r Sys.Date()`"
weight : 5
chapter : false
pre : " <b> 2.1.5 </b> "
---


**Goal**  
Confirm that **AWS Config** is evaluating your environment and that you can **see, filter, and export** current **NON_COMPLIANT** resources for triage.

> **Prerequisites:** 2.1.1 (S3 bucket), 2.1.2 (Enable Config), 2.1.3 (Rules), 2.1.4 (Review setup).

---

## What to check

- **Recorder** is **On** and snapshots/history are delivered to S3.  
- **Rules** show recent **Last successful evaluation** timestamps.  
- **Compliance summary** exists (counts by rule and by resource type).  
- You can **list NON_COMPLIANT resources** and export them.

---

## A) Verify in Console

1. **AWS Config → Rules**  
   - Column **Compliance** should show counts (Compliant / Noncompliant).  
   - Open a rule (e.g., **RESTRICTED_SSH**) → check **Last successful evaluation** and **Recently active resources**.
2. **AWS Config → Compliance → Summary**  
   - View **By Config rule** and **By resource type**.
3. **AWS Config → Resources**  
   - Filter **Compliance status = Noncompliant** and **Resource type** (e.g., *AWS::S3::Bucket*, *AWS::EC2::SecurityGroup*).  
   - Export if needed.

📸 Upload later:
- `/images/2-1-5-rules-summary.png` *(Rules list with compliance counts)*  
- `/images/2-1-5-rule-details.png` *(Rule details with last evaluation)*  
- `/images/2-1-5-compliance-summary.png` *(Compliance summary: by rule/type)*  
- `/images/2-1-5-noncompliant-filter.png` *(Resources filtered = Noncompliant)*

---

## B) Verify via CLI (quick summaries)

```bash
REGION="ap-southeast-1"

# 1) Compliance summary by rule (counts of compliant/noncompliant)
aws configservice get-compliance-summary-by-config-rule --region "$REGION"

# 2) Compliance summary by resource type (focus on networking)
aws configservice get-compliance-summary-by-resource-type \
  --region "$REGION" \
  --resource-types \
    AWS::EC2::VPC AWS::EC2::Subnet AWS::EC2::SecurityGroup AWS::EC2::NetworkAcl \
    AWS::EC2::InternetGateway AWS::EC2::NatGateway AWS::EC2::VPCEndpoint \
    AWS::ElasticLoadBalancingV2::LoadBalancer AWS::S3::Bucket

# 3) List your rules (names) to iterate if needed
aws configservice describe-config-rules --region "$REGION" \
  --query 'ConfigRules[].ConfigRuleName' --output table
```

## C) Pull NON_COMPLIANT resources per rule
```bash

RULE="s3-bucket-encryption-required"   # example from 2.1.3
aws configservice get-compliance-details-by-config-rule \
  --region "$REGION" \
  --config-rule-name "$RULE" \
  --compliance-types NON_COMPLIANT \
  --query 'EvaluationResults[].EvaluationResultIdentifier.EvaluationResultQualifier.[ResourceType,ResourceId]' \
  --output table
```
**Export all NON_COMPLIANT across rules (simple loop):**


```bash

for R in $(aws configservice describe-config-rules --region "$REGION" \
         --query 'ConfigRules[].ConfigRuleName' --output text); do
  echo "=== $R ==="
  aws configservice get-compliance-details-by-config-rule \
    --region "$REGION" \
    --config-rule-name "$R" \
    --compliance-types NON_COMPLIANT \
    --query 'EvaluationResults[].EvaluationResultIdentifier.EvaluationResultQualifier.[ResourceType,ResourceId]' \
    --output table
done
```

## D) (Optional) If you created a Config Aggregator
Summarize compliance across accounts/Regions in your home Region:

```bash
HOME_REGION="ap-southeast-1"
AGGR_NAME="network-aggregator"

# By rule, aggregated
aws configservice describe-aggregate-compliance-by-config-rules \
  --region "$HOME_REGION" \
  --configuration-aggregator-name "$AGGR_NAME" \
  --max-results 50

# Drill into one rule's NON_COMPLIANT resources
aws configservice get-aggregate-compliance-details-by-config-rule \
  --region "$HOME_REGION" \
  --configuration-aggregator-name "$AGGR_NAME" \
  --config-rule-name "restricted-ssh" \
  --compliance-type NON_COMPLIANT \
  --limit 50
```
📸 Upload later:

/images/2-1-5-aggregator-summary.png (Aggregator: compliance by rule)

## E) Evidence in S3 (spot-check)
**Confirm objects are landing in your audit bucket:**

```bash

ACCOUNT_ID="$(aws sts get-caller-identity --query Account --output text)"
BUCKET="network-compliance-logs-$ACCOUNT_ID-$REGION"

aws s3 ls "s3://$BUCKET/config-logs/AWSLogs/$ACCOUNT_ID/Config/$REGION/" --recursive | head -n 20
```
You should see configuration history and snapshot files (JSON/Compressed) for recorded resources.

## F) Troubleshooting
**All rules** say Evaluating or No results

Recorder may be off, or rules were just created — wait a few minutes and refresh.

Some managed rules require Input parameters (e.g., allowed ports). Edit the rule and set them.

**0 noncompliant but you expect violations**

Ensure the resource types are recorded (2.1.2).

Check rule scope/parameters (the rule may not target your resources).

For network checks (e.g., Flow Logs), verify the prerequisite feature exists to evaluate against.

**S3 AccessDenied / missing files**

Recheck bucket policy from 2.1.1 and KMS key policy if using SSE-KMS.





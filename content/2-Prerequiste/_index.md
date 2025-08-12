---
title : "Configure Monitoring Environment "
date : "`r Sys.Date()`"
weight : 2
chapter : false
pre : " <b> 2. </b> "
---

**Goal**  
Sanity-check the monitoring stack so continuous compliance works end-to-end: **AWS Config** recording → evidence in **S3**, **CloudTrail** logging, **Security Hub** enabled with standards, and required **IAM roles** in place.

**You will validate**
- AWS Config **recorder** and **delivery** (to S3 `config-logs/`)
- **CloudTrail** writing multi-Region logs to S3 `cloudtrail-logs/` (+ validation)
- **Security Hub** enabled and standards (FSBP/CIS/PCI) active
- **IAM roles** exist (Config SLR, Lambda remediation, QuickSight, optional SSM Automation)
- S3 bucket **encryption**, **versioning**, and **public access block**

---

## A) Verify in Console

1) **AWS Config → Settings**  
   - Recorder = **On**, **All supported resources** *(or your chosen network types)*  
   - Delivery channel → **S3 bucket** = `network-compliance-logs-<account>-<region>`, prefix `config-logs/`

2) **Amazon S3 → logs bucket**  
   - Browse to `config-logs/AWSLogs/<ACCOUNT_ID>/Config/<region>/...` **and**  
     `cloudtrail-logs/AWSLogs/<ACCOUNT_ID>/CloudTrail/<region>/...` — objects present  
   - **Bucket** shows **Versioning = Enabled**, **Default encryption** = **SSE-S3 or SSE-KMS**, **Block public access = ON**

3) **CloudTrail → Trails**  
   - Trail (e.g., `network-compliance-trail`) = **Logging**, **Multi-Region = Yes**, **Log file validation = Enabled**

4) **Security Hub**  
   - **Enabled** in this Region  
   - **Standards**: FSBP / CIS / PCI DSS show **Enabled** and controls evaluating  
   - (Optional) **Finding aggregation** configured to your home Region

5) **IAM → Roles**  
   - `AWSServiceRoleForConfig` exists  
   - `LambdaRemediationRole` (or your custom name) exists  
   - `aws-quicksight-service-role-v0` attached with your **QuickSightAthenaAccess** policy  
   - *(Optional)* `ConfigRemediationAutomationRole` (for SSM runbooks)

📸 Upload later:
- `/images/2-2-config-settings.png`
- `/images/2-2-s3-keys.png`
- `/images/2-2-cloudtrail-status.png`
- `/images/2-2-sh-standards-enabled.png`
- `/images/2-2-iam-roles.png`

---

## B) CLI quick verification

```bash
# --- Set your context ---
REGION="ap-southeast-1"
ACCOUNT_ID="$(aws sts get-caller-identity --query Account --output text)"
BUCKET="network-compliance-logs-$ACCOUNT_ID-$REGION"
TRAIL="network-compliance-trail"

echo "== AWS Config =="
aws configservice describe-configuration-recorder-status   --region "$REGION"
aws configservice describe-delivery-channel-status         --region "$REGION"
echo "S3 (Config keys)"; aws s3 ls "s3://$BUCKET/config-logs/AWSLogs/$ACCOUNT_ID/Config/$REGION/" --recursive | head -n 10

echo "== CloudTrail =="
aws cloudtrail get-trail-status --name "$TRAIL" --region "$REGION"
echo "S3 (CloudTrail keys)"; aws s3 ls "s3://$BUCKET/cloudtrail-logs/AWSLogs/$ACCOUNT_ID/" --recursive | head -n 10

echo "== Security Hub =="
aws securityhub get-enabled-standards --region "$REGION"
aws securityhub get-findings --region "$REGION" --max-results 5 \
  --filters '{"RecordState":[{"Comparison":"EQUALS","Value":"ACTIVE"}]}' >/dev/null && echo "Findings reachable ✔"

echo "== IAM roles =="
for R in AWSServiceRoleForConfig LambdaRemediationRole aws-quicksight-service-role-v0 ConfigRemediationAutomationRole; do
  aws iam get-role --role-name "$R" >/dev/null 2>&1 && echo "Role $R ✔" || echo "Role $R (optional/missing)"
done

echo "== S3 security =="
aws s3api get-bucket-encryption --bucket "$BUCKET"
aws s3api get-bucket-versioning --bucket "$BUCKET"
aws s3api get-public-access-block --bucket "$BUCKET"
```

## C) “All-green” checklist (what “good” looks like)
describe-configuration-recorder-status → recording: true

describe-delivery-channel-status → recent lastSuccessful* timestamps, no errors

get-trail-status → IsLogging: true, LatestDeliveryTime recent, LogFileValidationEnabled: true

get-enabled-standards → at least FSBP (and optional CIS/PCI) listed

S3 encryption details returned, versioning = Enabled, public access block shows all four = true

## D) Common issues & quick fixes
**Config shows Recording = false → start it:**

```bash

aws configservice start-configuration-recorder --region "$REGION" --configuration-recorder-name default
```
**S3 AccessDenied for Config/CloudTrail** → re-apply bucket policy from 2.1.1; if SSE-KMS, ensure KMS key policy allows the service.

**No CloudTrail keys** → start-logging the trail and confirm trail points to the correct bucket/prefix.

**Security Hub standards not evaluating** → enable standards in 3.2.2 and wait a few minutes; verify AWS Config is recording the required resource types.

**IAM role missing** → create from 2.1.6 snippets (Config SLR, Lambda, QuickSight, SSM Automation).

## E) Good practices
Keep **All supported resources** in Config to catch dependencies beyond networking.

Tag rules and packs with Compliance=Network for reporting slices.

In multi-account setups, add a **Config Aggregator** in a home Region (optional) before moving to data collection.

+++
title = "Clean up resources"
date = 2022
weight = 6
chapter = false
pre = "<b>6. </b>"
+++

**Goal**  
Remove resources created for the lab to avoid ongoing charges. Keep any evidence/logs you need for audit.

> **Warning**  
> Don’t delete logs you intend to keep. If in doubt, **archive** to Glacier instead of deleting.

---

## 6.1 Disable automations and delete functions

**Console**
- **EventBridge**: disable/delete rules you created for auto-remediation.  
- **Lambda**: delete remediation functions.

**CLI (example)**
```bash
REGION=ap-southeast-1
RULE_NAME=auto-remediate-s3-encryption
LAMBDA_FN=auto-remediate-s3

aws events remove-targets --region "$REGION" --rule "$RULE_NAME" --ids "0"
aws events delete-rule   --region "$REGION" --name "$RULE_NAME"
aws lambda delete-function --region "$REGION" --function-name "$LAMBDA_FN"
```

## 6.2 Remove Config remediation (if configured)
**Console**: AWS Config → **Rules** → select rule → **Remediation** → **Edit** → disable/delete configuration.

CLI

```bash

aws configservice delete-remediation-configuration \
  --region "$REGION" \
  --config-rule-name "S3_BUCKET_SERVER_SIDE_ENCRYPTION_ENABLED"
```

## 6.3 Disable Security Hub (optional for lab)
```bash

aws securityhub disable-security-hub --region "$REGION"
```
(If using multi-account/aggregation, disable from the delegated/admin account first.)


# 6.4 Stop and delete CloudTrail (lab trail)
```bash

TRAIL="network-compliance-trail"
aws cloudtrail stop-logging --name "$TRAIL" --region "$REGION"
aws cloudtrail delete-trail  --name "$TRAIL" --region "$REGION"
```

## 6.5 Empty and delete the logs bucket (optional!)
Only do this if you don’t need the audit evidence.

```bash

ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
BUCKET="network-compliance-logs-$ACCOUNT_ID-$REGION"

# Purge all versions (if versioning is enabled)
aws s3api delete-objects --bucket "$BUCKET" --delete \
  "$(aws s3api list-object-versions --bucket "$BUCKET" \
  --query='{Objects: Versions[].{Key:Key,VersionId:VersionId}}' --output=json)" || true
aws s3api delete-objects --bucket "$BUCKET" --delete \
  "$(aws s3api list-object-versions --bucket "$BUCKET" \
  --query='{Objects: DeleteMarkers[].{Key:Key,VersionId:VersionId}}' --output=json)" || true

# Delete the bucket
aws s3api delete-bucket --bucket "$BUCKET" --region "$REGION"
```

# 6.6 Remove IAM roles/policies used for the lab
Detach and delete policies you created for:

**ConfigRemediationAutomationRole** (SSM Automation)

**Lambda execution** role(s)

Then delete the roles themselves.

# 6.7 (Optional) Delete Glue/Athena/QuickSight artifacts
**Glue:** delete Crawlers, Database, and Tables created for the lab.

**Athena:** clear query results in the S3 athena-results/ prefix if desired.

**QuickSight:** delete datasets/dashboards; remove S3/Athena/Glue permissions.


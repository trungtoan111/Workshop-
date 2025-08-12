---
title : "Execute Auto-Remediation"
date :  "`r Sys.Date()`" 
weight : 5 
chapter : false
pre : " <b> 5. </b> "
---

**Goal**  
Automatically fix common misconfigurations the moment they’re detected. You’ll set up **native AWS Config remediation with SSM Automation (recommended)**, and an alternative **EventBridge → Lambda** path for custom fixes. All actions are logged for audit.

> **Prerequisites**
> - Section 2 is complete (S3 logs bucket, CloudTrail, Config recording).
> - Security Hub standards (FSBP / CIS / PCI) are enabled (Section 3.2.2).
> - You have an **IAM role** that SSM Automation can assume to make changes (see below).

### Architecture
![Auto-remediation flow: Config → (SSM Automation or EventBridge→Lambda) → Logs/CloudTrail](/images/5-arch-remediation.png)

---

## 5.1 Choose what to auto-remediate

Start with low-risk, high-signal fixes:
- **S3**: enforce bucket encryption; block public access.
- **Security Groups**: remove `0.0.0.0/0` on SSH(22) / RDP(3389).
- **VPC Flow Logs**: ensure enabled at VPC/Subnet/ENI level.
- **CloudTrail**: keep multi-Region & log file validation ON.

> Tip: Keep “detect only” for impactful changes (e.g., database/networking in prod) until change control is in place.

---

## 5.2 Native: AWS Config Remediation with SSM Automation (recommended)

**Why**: first-class integration, retries/backoff, clear audit in Config + CloudTrail.

### A) Create/verify the **AutomationAssumeRole** (used by SSM Automation)
This role performs the actual changes during remediation.

**Policy sketch (adjust to your scope):**
```json
{
  "Version":"2012-10-17",
  "Statement":[
    { "Effect":"Allow","Action":["s3:PutBucketEncryption","s3:PutBucketAcl","s3:PutBucketPolicy"],"Resource":"*" },
    { "Effect":"Allow","Action":["ec2:RevokeSecurityGroupIngress","ec2:DescribeSecurityGroups"],"Resource":"*" },
    { "Effect":"Allow","Action":["logs:CreateLogGroup","logs:CreateLogStream","logs:PutLogEvents"],"Resource":"*" }
  ]
}

```
Attach this policy to a role (e.g., ConfigRemediationAutomationRole) that SSM can assume.

## B) Associate a remediation to a Config rule
Example: for **S3 bucket encryption** (rule: S3_BUCKET_SERVER_SIDE_ENCRYPTION_ENABLED) use the managed runbook AWS-ConfigureS3BucketEncryption.

**Find runbook parameters first:**

```bash

aws ssm describe-document --name AWS-ConfigureS3BucketEncryption --query 'Document.Parameters'
Wire the remediation (automatic)
```

```bash

REGION=ap-southeast-1
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)

aws configservice put-remediation-configurations \
  --region "$REGION" \
  --remediation-configurations '[
    {
      "ConfigRuleName":"S3_BUCKET_SERVER_SIDE_ENCRYPTION_ENABLED",
      "TargetType":"SSM_DOCUMENT",
      "TargetId":"AWS-ConfigureS3BucketEncryption",
      "Automatic":true,
      "RetryAttemptSeconds":60,
      "MaximumAutomaticAttempts":3,
      "Parameters":{
        "AutomationAssumeRole":{"StaticValue":{"Values":["arn:aws:iam::'"$ACCOUNT_ID"':role/ConfigRemediationAutomationRole"]}},
        "BucketName":{"ResourceValue":{"Value":"RESOURCE_ID"}},
        "SSEAlgorithm":{"StaticValue":{"Values":["AES256"]}}
      }
    }
  ]'
```

Notes

Use RESOURCE_ID/RESOURCE_TYPE mappings for parameters that come from the non-compliant resource.

Repeat for other rules (e.g., a remediation runbook to restrict SG ingress).

Results appear under AWS Config → Rules → (rule) → Remediation history.

📸 Upload later:

/images/5-2-config-remediation-wire.png (Attach remediation to rule)

/images/5-2-remediation-history.png (Remediation history & status)

## 5.3 Alternative: EventBridge → Lambda (for custom fixes)
Use when you need custom logic beyond the managed runbooks.

## A) Event pattern (Config non-compliant)
```json

{
  "source": ["aws.config"],
  "detail-type": ["Config Rules Compliance Change"],
  "detail": {
    "configRuleName": ["S3_BUCKET_SERVER_SIDE_ENCRYPTION_ENABLED"],
    "newEvaluationResult": { "complianceType": ["NON_COMPLIANT"] }
  }
}
```

## B) Sample Lambda (Python 3.x): enforce S3 encryption
```python

import os, json, boto3

s3 = boto3.client("s3")

def lambda_handler(event, context):
    # Config event → resourceId is the bucket name for S3 buckets
    detail = event.get("detail", {})
    resource_id = (detail.get("resourceId") or
                   detail.get("newEvaluationResult", {}).get("evaluationResultIdentifier", {})
                        .get("evaluationResultQualifier", {}).get("resourceId"))
    if not resource_id:
        return {"status": "skipped", "reason": "no resourceId"}

    s3.put_bucket_encryption(
        Bucket=resource_id,
        ServerSideEncryptionConfiguration={
            "Rules":[{"ApplyServerSideEncryptionByDefault":{"SSEAlgorithm":"AES256"}}]
        }
    )
    return {"status":"remediated","bucket":resource_id}
```

**Execution role (minimal):**

s3:PutBucketEncryption on target buckets

CloudWatch Logs permissions for logging

📸 Upload later:

/images/5-3-eb-rule.png (EventBridge rule)

/images/5-3-lambda-logs.png (Lambda invocation logs)

## 5.4 Log & surface remediation
**CloudTrail** records the API calls performed by SSM/Lambda (e.g., PutBucketEncryption, RevokeSecurityGroupIngress).

**AWS Config** shows the Remediation history and status per resource.

**Security Hub findings** auto-update on next evaluation; you can also set Workflow status via API if you manage state.

**(Optional) Mark related Security Hub finding as RESOLVED**

```bash

# Example sketch: filter finding(s) and update workflow status
aws securityhub batch-update-findings \
  --region "$REGION" \
  --finding-identifiers Id="<FINDING_ID>",ProductArn="<PRODUCT_ARN>" \
  --workflow Status="RESOLVED" \
  --note Text="Auto-remediated by Config/SSM",UpdatedBy="automation"
```

## 5.5 Test the flow
Create a **non-compliant** resource (e.g., S3 bucket without encryption).

Wait for Config evaluation → **NON_COMPLIANT**.

Observe remediation (SSM Automation or Lambda) apply the fix.

Verify:

S3 now has **Default Encryption**

Config rule back to **COMPLIANT**

CloudTrail logs the change; Security Hub updates findings.

📸 Upload later:

/images/5-5-test-before.png (Bucket w/o encryption)

/images/5-5-test-after.png (Encryption enforced + rule compliant)


---
title   : "Create IAM Roles and Policies"
date    : "`r Sys.Date()`"
weight  : 1
chapter : false
pre     : " <b> 2.1.6 </b> "
---

**Goal**  
Create/verify IAM roles so services can record, remediate, and report compliance:

- **AWS Config** service-linked role (record/evaluate)
- **Lambda Remediation** role (for custom auto-fixes)
- **QuickSight** access to **Athena/Glue/S3** (reporting)
- *(Optional)* **SSM Automation assume role** for Config native remediation (Section 5)

> **Prerequisites:** 2.1.1–2.1.5 completed; you have IAM permissions to create roles/policies.

---

## A) AWS Config service-linked role (verify/create)

**Console**
1. IAM → **Roles** → search **`AWSServiceRoleForConfig`**.  
2. If present, no action. If missing, create it.

**CLI**
```bash
aws iam create-service-linked-role --aws-service-name config.amazonaws.com || true
aws iam get-role --role-name AWSServiceRoleForConfig
```
📸 Upload later:

/images/2-1-6-config-slr.png

## B) Lambda Remediation role (EventBridge → Lambda path)
**1) Trust policy**

```json

{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": { "Service": "lambda.amazonaws.com" },
    "Action": "sts:AssumeRole"
  }]
}
```
## 2) Permissions policy (start least-privilege; narrow later)

```json

{
  "Version": "2012-10-17",
  "Statement": [
    { "Sid": "S3Encryption",
      "Effect": "Allow",
      "Action": [ "s3:PutBucketEncryption", "s3:GetBucketEncryption" ],
      "Resource": "arn:aws:s3:::*"
    },
    { "Sid": "RestrictIngress",
      "Effect": "Allow",
      "Action": [ "ec2:DescribeSecurityGroups", "ec2:RevokeSecurityGroupIngress" ],
      "Resource": "*"
    },
    { "Sid": "CloudWatchLogs",
      "Effect": "Allow",
      "Action": [ "logs:CreateLogGroup", "logs:CreateLogStream", "logs:PutLogEvents" ],
      "Resource": "*"
    },
    { "Sid": "OptionalUpdateFindings",
      "Effect": "Allow",
      "Action": [ "securityhub:BatchUpdateFindings" ],
      "Resource": "*"
    }
  ]
}
```
**CLI**  

```bash

ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
REGION=ap-southeast-1

# Create role
aws iam create-role \
  --role-name LambdaRemediationRole \
  --assume-role-policy-document file://lambda-trust.json

# Attach inline policy
aws iam put-role-policy \
  --role-name LambdaRemediationRole \
  --policy-name LambdaRemediationActions \
  --policy-document file://lambda-remediation-policy.json

# (Optional) Use AWS managed logging policy
aws iam attach-role-policy \
  --role-name LambdaRemediationRole \
  --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
```
📸 Upload later:

/images/2-1-6-lambda-role.png

## C) QuickSight access for Athena/Glue/S3 (reporting)
**Policy to allow Athena/Glue + read logs S3 + write Athena results**

Replace NETWORK_COMPLIANCE_BUCKET (and restrict prefixes as needed).

```json

{
  "Version": "2012-10-17",
  "Statement": [
    { "Sid": "AthenaAccess",
      "Effect": "Allow",
      "Action": [
        "athena:StartQueryExecution","athena:GetQueryExecution","athena:GetQueryResults",
        "athena:ListDataCatalogs","athena:ListWorkGroups","athena:GetWorkGroup"
      ],
      "Resource": "*"
    },
    { "Sid": "GlueCatalogRead",
      "Effect": "Allow",
      "Action": [
        "glue:GetDatabase","glue:GetDatabases","glue:GetTable","glue:GetTables",
        "glue:GetPartition","glue:GetPartitions"
      ],
      "Resource": "*"
    },
    { "Sid": "S3ReadLogs",
      "Effect": "Allow",
      "Action": [ "s3:GetObject", "s3:ListBucket" ],
      "Resource": [
        "arn:aws:s3:::NETWORK_COMPLIANCE_BUCKET",
        "arn:aws:s3:::NETWORK_COMPLIANCE_BUCKET/*"
      ]
    },
    { "Sid": "S3AthenaResults",
      "Effect": "Allow",
      "Action": [ "s3:PutObject", "s3:GetObject", "s3:ListBucket" ],
      "Resource": [
        "arn:aws:s3:::NETWORK_COMPLIANCE_BUCKET",
        "arn:aws:s3:::NETWORK_COMPLIANCE_BUCKET/athena-results/*"
      ]
    }
  ]
}
```
**Attach to QuickSight service role**

```bash

# Role name is standard:
aws iam get-role --role-name aws-quicksight-service-role-v0

# Create & attach
aws iam create-policy \
  --policy-name QuickSightAthenaAccess \
  --policy-document file://quicksight-athena-policy.json

POLICY_ARN=$(aws iam list-policies \
  --query "Policies[?PolicyName=='QuickSightAthenaAccess'].Arn" --output text)

aws iam attach-role-policy \
  --role-name aws-quicksight-service-role-v0 \
  --policy-arn "$POLICY_ARN"
```
**Console alternative**: QuickSight → Manage QuickSight → Security & permissions → grant Athena, Glue, and your S3 bucket.

📸 Upload later:

/images/2-1-6-quicksight-role.png

/images/2-1-6-quicksight-perms.png

## D) (Optional) SSM Automation assume role (for Config native remediation)
Trust policy

```json

{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": { "Service": "ssm.amazonaws.com" },
    "Action": "sts:AssumeRole"
  }]
}
Permissions policy (sample; narrow later)

json
Sao chép
Chỉnh sửa
{
  "Version": "2012-10-17",
  "Statement": [
    { "Effect": "Allow", "Action": [ "s3:PutBucketEncryption","s3:GetBucketEncryption" ], "Resource": "arn:aws:s3:::*" },
    { "Effect": "Allow", "Action": [ "ec2:DescribeSecurityGroups","ec2:RevokeSecurityGroupIngress" ], "Resource": "*" },
    { "Effect": "Allow", "Action": [ "logs:CreateLogGroup","logs:CreateLogStream","logs:PutLogEvents" ], "Resource": "*" }
  ]
}
```
**CLI**

```bash

aws iam create-role \
  --role-name ConfigRemediationAutomationRole \
  --assume-role-policy-document file://ssm-automation-trust.json

aws iam put-role-policy \
  --role-name ConfigRemediationAutomationRole \
  --policy-name SsmAutomationActions \
  --policy-document file://ssm-automation-policy.json
```
📸 Upload later:

/images/2-1-6-ssm-automation-role.png

## E) Validate
```bash

aws iam get-role --role-name AWSServiceRoleForConfig
aws iam get-role --role-name LambdaRemediationRole
aws iam get-role --role-name aws-quicksight-service-role-v0
aws iam get-role --role-name ConfigRemediationAutomationRole  # if created
```

Test a small Lambda using **LambdaRemediationRole** and confirm CloudWatch Logs entries.

In **QuickSight**, create an Athena dataset to verify access to **Glue Catalog and S3.**

**Good practices**
Scope actions to ARNs/tags as soon as targets are known (avoid "Resource": "*")

Use separate roles per automation to limit blast radius

Prefer customer-managed policies over large inline blobs for versioning/reuse

With SSE-KMS buckets, ensure KMS key policies include these roles (Lambda/SSM/QuickSight)

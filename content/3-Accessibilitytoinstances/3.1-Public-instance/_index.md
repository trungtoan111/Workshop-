---
title   : "Connect AWS Config to Network Resources"
date    : "`r Sys.Date()`"
weight  : 1
chapter : false
pre     : " <b> 3.1 </b> "
---

**Goal**  
Ensure **AWS Config** is recording **all relevant network resources**, delivering snapshots to your **S3 audit bucket**, and is ready to evaluate networking controls (used by Security Hub standards and Config rules).

> **Prerequisites**
> - You created the **logs S3 bucket** (e.g., `network-compliance-logs-<account>-<region>`) with **versioning & encryption**.
> - AWS Config is available in this Region (Section 2).
> - You have permissions to configure Config (`config:*`) and S3.

---

## What you will set up

- **Recording scope** that includes core **network resource types**  
  *(recommended: record all supported resource types; otherwise ensure at minimum:)*  
  `AWS::EC2::VPC, AWS::EC2::Subnet, AWS::EC2::RouteTable, AWS::EC2::InternetGateway, AWS::EC2::NatGateway, AWS::EC2::VPCEndpoint, AWS::EC2::SecurityGroup, AWS::EC2::NetworkAcl, AWS::EC2::NetworkInterface, AWS::ElasticLoadBalancingV2::LoadBalancer, AWS::ElasticLoadBalancingV2::Listener, AWS::ElasticLoadBalancingV2::TargetGroup, AWS::EC2::EIP, AWS::S3::Bucket`.

- **Delivery channel** pointing to your S3 bucket under `config-logs/`.

- *(Optional)* A **Config Aggregator** (multi-account/Region) to centralize inventory/compliance.

---

## A) Configure recording & delivery (Console)

1. Open **AWS Config → Settings**.  
2. **Configuration recorder**: ensure **On**.  
3. **Recorded resources**:  
   - **Record all supported resources** *(recommended)*, or  
   - **Specify** only the types listed above for networking.  
4. **Delivery channel**:  
   - **S3 bucket**: `network-compliance-logs-<account>-<region>`  
   - **S3 prefix**: `config-logs/`  
   - *(Optional)* **KMS key** for SSE-KMS.  
5. **Save** and confirm status shows **Recording**.

📸 Upload later:
- `/images/3-1-config-settings.png` *(Recorder = On, bucket set)*  
- `/images/3-1-recorded-resources.png` *(Recorded resource types)*  
- `/images/3-1-config-bucket.png` *(Bucket/prefix selection)*

> **Note:** Keep **include global resource types** = **off** for a network-focused lab (IAM, etc., are global). You can enable later if needed.

---

## B) Verify snapshots & evaluation start

- After a few minutes, check **S3**: you should see objects under `config-logs/AWSLogs/<ACCOUNT_ID>/Config/<region>/...`  
- In **AWS Config → Resource inventory**, filter by **VPC**, **Subnet**, **SecurityGroup**, etc., to ensure resources are recorded.  
- If you already created rules/conformance packs, go to **Rules → Compliance** and confirm evaluations are running.

📸 Upload later:
- `/images/3-1-eval-status.png` *(Resources visible / evaluations progressing)*

---

## C) Configure recording via CLI (optional)

```bash
REGION=ap-southeast-1
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
BUCKET="network-compliance-logs-$ACCOUNT_ID-$REGION"

# 1) Put/Update the configuration recorder (record ALL supported; no global types)
aws configservice put-configuration-recorder --region "$REGION" --configuration-recorder "{
  \"name\": \"default\",
  \"roleARN\": \"arn:aws:iam::$ACCOUNT_ID:role/<ConfigRole>\",
  \"recordingGroup\": {
    \"allSupported\": true,
    \"includeGlobalResourceTypes\": false
  }
}"

# 2) Set the delivery channel (S3 bucket/prefix)
aws configservice put-delivery-channel --region "$REGION" --delivery-channel "{
  \"name\": \"default\",
  \"s3BucketName\": \"$BUCKET\",
  \"s3KeyPrefix\": \"config-logs\"
}"

# 3) Start the recorder (if not already running)
aws configservice start-configuration-recorder --region "$REGION" --configuration-recorder-name default

# 4) Quick status check
aws configservice describe-configuration-recorder-status --region "$REGION"
```

## D) (Optional) Create a Config Aggregator (multi-account/Region)
Aggregate inventory and compliance into a **home Region.**

**Console**

**AWS Config → Aggregators → Create aggregator**

Choose **Organization** (if using AWS Organizations) or **Specific accounts.**

Select **All current and future Regions (recommended).**

**Create** and wait for initial aggregation.

CLI (accounts list example)

```bash

HOME_REGION=ap-southeast-1
aws configservice put-configuration-aggregator \
  --region "$HOME_REGION" \
  --configuration-aggregator-name "network-aggregator" \
  --account-aggregation-sources "AccountIds=[\"111111111111\",\"222222222222\"],AllAwsRegions=true"
```
📸 Upload later:

/images/3-1-aggregator.png (Aggregator status: HEALTHY, Regions/accounts listed)

## E) Good practices
**Tag** your rules/packs with Compliance=Network for easier reporting.

Keep the **S3 bucket** private, versioned, and encrypted (SSE-KMS if available).

Pair recording with relevant managed rules (e.g., **RESTRICTED_SSH, VPC_FLOW_LOGS_ENABLED, EIP_ATTACHED)** and/or a **Conformance Pack** for network baselines.

In multi-account setups, consider **delegated admin** for Security Hub/Config and use the **Aggregator** for a single pane of glass.



---
title : "Connect and Collect Compliance Data"
date : "`r Sys.Date()`"
weight : 3
chapter : false
pre : " <b> 3. </b> "
---
**Goal**  
Wire up **AWS Config** and **AWS Security Hub** to your **network resources**, enable the right **security standards**, add **private SSM connectivity** (VPC endpoints), and **connect to a private instance** so compliance evaluations and findings start flowing.

> **Prerequisites**
> - Section **2.1.x** completed (S3 logs bucket, AWS Config recording, initial rules, CloudTrail, IAM roles).
> - You have permissions for **Config**, **Security Hub**, **EC2/VPC**, and **SSM**.

---

### Architecture

![Connect & Collect – Config + Security Hub + SSM endpoints](/images/3-arch-connect-collect.png)

- **AWS Config** records VPC/SG/NACL/ELB/S3, evaluates rules → evidence to **S3**.  
- **Security Hub** aggregates controls (FSBP/CIS/PCI) → unified **findings**.  
- **SSM VPC Interface Endpoints** (`ssm`, `ssmmessages`, `ec2messages`) let you manage **private instances** without internet.

---

## What you will set up in Section 3

1) **Connect AWS Config to Network Resources** (ensure recording scope & delivery)  
2) **Integrate Security Hub with AWS Config** and **enable standards** (FSBP/CIS/PCI)  
3) **Create SSM VPC Interface Endpoints** and **connect to a private instance** via Session Manager

---

## Content

- **[3.1 Connect AWS Config to Network Resources](./3.1-connect-aws-config-to-network-resources/)**  
- **[3.2 Integrate Security Hub with AWS Config](./3.2-integrate-security-hub-with-aws-config/)**  
  - [3.2.1 Enable Security Hub](./3.2.1-enable-security-hub/)  
  - [3.2.2 Enable Security Standards](./3.2.2-enable-standards/)  
  - [3.2.2.1 Enable FSBP](./3.2.2.1-enable-aws-fsbp/)  
  - [3.2.2.2 Enable PCI DSS](./3.2.2.2-enable-pci/)  
  - [3.2.2.3 Create Endpoint ec2messages](./3.2.2.3-create-endpoint-ec2messages/)  
- **[3.2.3 Connect to Instance (Session Manager)](./3.2.3-connect-to-instance/)**

📸 Upload later:
- `/images/3-arch-connect-collect.png`

---

## Quick check: data is flowing

After completing 3.1–3.2:

```bash
REGION=ap-southeast-1

# Config is recording & delivering
aws configservice describe-configuration-recorder-status --region "$REGION"
aws configservice describe-delivery-channel-status --region "$REGION"

# Security Hub standards enabled & a few findings visible
aws securityhub get-enabled-standards --region "$REGION"
aws securityhub get-findings --region "$REGION" --max-results 5 \
  --filters '{"RecordState":[{"Comparison":"EQUALS","Value":"ACTIVE"}]}'



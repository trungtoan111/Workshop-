---
title : "PCI-DSS"
date : "`r Sys.Date()`"
weight : 2
chapter : false
pre : " <b> 3.2.2.2 </b> "
---


**Goal**  
Enable **PCI DSS** in Security Hub so controls relevant to cardholder data environments (CDE) are evaluated and surfaced as **findings** (e.g., logging, network segmentation, encryption, access control).

> **Prerequisite:** You completed **3.2.1 Enable Security Hub**.  
> **Scope note:** Only enable PCI DSS if your workload **stores/processes/transmits** cardholder data.

---

## Console

1. Open **Security Hub → Standards**.  
2. Locate **PCI DSS** → click **Enable**.  
3. Choose **Enable all controls** *(recommended)*, or **Customize** to disable controls that don’t apply.  
4. Wait for evaluation to start (*status: Evaluating → Passed/Failed*). Findings will appear after the first run.

📸 Upload later:
- `/images/3-2-2-2-pci-list.png` *(PCI DSS in Standards list)*
- `/images/3-2-2-2-pci-enable-dialog.png` *(Enable dialog)*
- `/images/3-2-2-2-pci-controls.png` *(PCI DSS controls view)*

> **Tips**
> - Ưu tiên các kiểm soát: **CloudTrail (management + data events)**, **S3 default encryption & Block Public Access**, **ALB/NLB TLS 1.2+**, **Security Group không mở 0.0.0.0/0**, **VPC Flow Logs**, **RDS/EBS encryption**.  
> - PCI đánh giá **theo Region**; lặp lại cho các Region bạn giám sát.  
> - Ghi rõ *disabled reason* nếu tắt một control (ví dụ vì môi trường không lưu trữ dữ liệu thẻ).

---

## CLI

```bash
REGION=ap-southeast-1

# 1) Discover available standards (and versions) in this Region
aws securityhub describe-standards --region "$REGION"

# 2) Set the PCI DSS ARN from your Region's output (version may differ!)
PCI_ARN="arn:aws:securityhub:${REGION}::standards/pci-dss/v/3.2.1"

# 3) Enable PCI DSS
aws securityhub batch-enable-standards \
  --region "$REGION" \
  --standards-subscription-requests StandardsArn="$PCI_ARN"

# 4) Confirm enabled standards
aws securityhub get-enabled-standards --region "$REGION"

# 5) (Optional) List PCI controls
SUB_ARN=$(aws securityhub get-enabled-standards --region "$REGION" \
  --query 'StandardsSubscriptions[?StandardsArn==`'"$PCI_ARN"'`].StandardsSubscriptionArn' \
  --output text)

aws securityhub describe-standards-controls \
  --region "$REGION" \
  --standards-subscription-arn "$SUB_ARN" \
  --max-results 50

# 6) (Optional) Disable a PCI control that doesn't apply (with justification)
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
aws securityhub update-standards-control \
  --region "$REGION" \
  --standards-control-arn "arn:aws:securityhub:${REGION}:${ACCOUNT_ID}:control/pci-dss/v/3.2.1/CONTROL_ID" \
  --control-status "DISABLED" \
  --disabled-reason "Out of scope for this environment"
```

---
title : "Enable standards"
date : "`r Sys.Date()`"
weight : 2
chapter : false
pre : " <b> 3.2.2 </b> "
---


**Goal**  
Enable **Security Hub standards** so controls (FSBP, CIS, PCI DSS) are evaluated and surfaced as unified **findings**.

> **Prerequisite:** **3.2.1** has enabled Security Hub in this Region.

---

## Console

1. Open **Security Hub → Standards**.  
2. Click **Enable standard** for the frameworks you need:
   - **AWS Foundational Security Best Practices (FSBP)**
   - **CIS AWS Foundations Benchmark**
   - **PCI DSS** *(if applicable to your workload)*
3. Choose **Enable all controls** *(recommended)*, or **Customize** to disable controls that don’t apply.
4. Click **Enable** and wait for evaluation to start (*status: Evaluating → Passed/Failed*).

📸 Upload later:
- `/images/3-2-sh-standards.png` *(Standards list with “Enabled” badges)*
- `/images/3-2-sh-enable-dialog.png` *(Enable standard dialog)*
- `/images/3-2-control-details.png` *(A control details page)*

> **Notes**
> - Mỗi Region bật tiêu chuẩn **riêng**; lặp lại ở các Region bạn giám sát.  
> - Chi phí dựa trên số lượng findings/ingestions.  
> - **SOC 2 / HIPAA** không phải standards native của Security Hub; map evidence qua **AWS Audit Manager** hoặc dùng **AWS Config Conformance Packs** nếu cần.

---

## CLI

```bash
REGION=ap-southeast-1

# 1) Discover available standards (and versions) in this Region
aws securityhub describe-standards --region "$REGION"

# 2) Set ARNs from your Region's output (versions may differ!)
FSBP_ARN="arn:aws:securityhub:${REGION}::standards/aws-foundational-security-best-practices/v/1.0.0"
CIS_ARN="arn:aws:securityhub:${REGION}::standards/cis-aws-foundations-benchmark/v/1.4.0"
PCI_ARN="arn:aws:securityhub:${REGION}::standards/pci-dss/v/3.2.1"   # enable only if needed

# 3) Enable standards
aws securityhub batch-enable-standards \
  --region "$REGION" \
  --standards-subscription-requests \
  StandardsArn="$FSBP_ARN" \
  StandardsArn="$CIS_ARN" \
  StandardsArn="$PCI_ARN"

# 4) (Optional) List controls for a standard
SUB_ARN=$(aws securityhub get-enabled-standards --region "$REGION" \
  --query 'StandardsSubscriptions[?StandardsArn==`'"$FSBP_ARN"'`].StandardsSubscriptionArn' \
  --output text)

aws securityhub describe-standards-controls \
  --region "$REGION" \
  --standards-subscription-arn "$SUB_ARN" \
  --max-results 20

# 5) (Optional) Disable a specific control if not applicable
#   Replace CONTROL_ID with an ID from describe-standards-controls output, e.g. "IAM.1"
aws securityhub update-standards-control \
  --region "$REGION" \
  --standards-control-arn "arn:aws:securityhub:${REGION}:${ACCOUNT_ID}:control/aws-foundational-security-best-practices/v/1.0.0/CONTROL_ID" \
  --control-status "DISABLED" \
  --disabled-reason "Not applicable to this environment"

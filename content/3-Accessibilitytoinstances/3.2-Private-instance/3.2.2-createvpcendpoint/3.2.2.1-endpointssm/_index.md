---
title : "SOC 2"
date : "`r Sys.Date()`"
weight : 1
chapter : false
pre : " <b> 3.2.2.1 </b> "
---

#### Enable AWS Foundational Security Best Practices (FSBP)

**Goal**  
Enable **AWS Foundational Security Best Practices (FSBP)** in Security Hub so baseline controls are continuously evaluated and surfaced as **findings**.

> **Prerequisite:** You completed **3.2.1 Enable Security Hub**.

---

## Console

1. Open **Security Hub → Standards**.  
2. Locate **AWS Foundational Security Best Practices (FSBP)** → click **Enable**.  
3. Choose **Enable all controls** *(recommended)*, or **Customize** to disable controls that don’t apply.  
4. Wait for evaluation to start (*status: Evaluating → Passed/Failed*). You’ll see findings appear shortly.

📸 Upload later:
- `/images/3-2-2-1-fsbp-list.png` *(FSBP in Standards list)*
- `/images/3-2-2-1-fsbp-enable-dialog.png` *(Enable dialog)*
- `/images/3-2-2-1-fsbp-controls.png` *(Controls view for FSBP)*

> **Notes**
> - Bật theo **Region**; lặp lại ở các Region bạn giám sát.  
> - Nhiều control FSBP dựa trên **AWS Config managed rules**.  
> - *(Khuyến nghị)* Trong **Security Hub → Settings → General**, bật **Auto-enable new controls** để tự động nhận các control mới trong tương lai.

---

## CLI

```bash
REGION=ap-southeast-1

# 1) Discover available standards (and versions) in this Region
aws securityhub describe-standards --region "$REGION"

# 2) Set FSBP ARN from your Region's output (version may differ!)
FSBP_ARN="arn:aws:securityhub:${REGION}::standards/aws-foundational-security-best-practices/v/1.0.0"

# 3) Enable FSBP
aws securityhub batch-enable-standards \
  --region "$REGION" \
  --standards-subscription-requests StandardsArn="$FSBP_ARN"

# 4) List your enabled standards to confirm
aws securityhub get-enabled-standards --region "$REGION"

# 5) (Optional) List controls under FSBP
SUB_ARN=$(aws securityhub get-enabled-standards --region "$REGION" \
  --query 'StandardsSubscriptions[?StandardsArn==`'"$FSBP_ARN"'`].StandardsSubscriptionArn' \
  --output text)

aws securityhub describe-standards-controls \
  --region "$REGION" \
  --standards-subscription-arn "$SUB_ARN" \
  --max-results 50

# 6) (Optional) Disable a specific control if not applicable
#    Replace CONTROL_ID with an ID from step 5 (e.g., "IAM.1", "S3.4", etc.)
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
aws securityhub update-standards-control \
  --region "$REGION" \
  --standards-control-arn "arn:aws:securityhub:${REGION}:${ACCOUNT_ID}:control/aws-foundational-security-best-practices/v/1.0.0/CONTROL_ID" \
  --control-status "DISABLED" \
  --disabled-reason "Not applicable to this environment"
```




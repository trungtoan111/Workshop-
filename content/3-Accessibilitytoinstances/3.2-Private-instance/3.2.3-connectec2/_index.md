---
title : "Connect to instance"
date : "`r Sys.Date()`"
weight : 3
chapter : false
pre : " <b> 3.2.3 </b> "
---


**Goal**  
Connect to a **private EC2 instance** without public IP or bastion, using **AWS Systems Manager Session Manager**. Traffic stays **inside the VPC** via the SSM **VPC Interface Endpoints**.

> **Prerequisites**
> - VPC Interface Endpoints created: `ssm`, `ssmmessages`, `ec2messages` (from 3.2.2.1–3.2.2.3).
> - Instance has **SSM Agent** (Amazon Linux 2 / Windows AMIs include it) and IAM role **AmazonSSMManagedInstanceCore**.
> - Endpoint Security Group allows **TCP 443** from your private subnets/VPC CIDR.
> - VPC **DNS resolution/hostnames** enabled; **Private DNS** enabled on endpoints.

---

## A) Connect via Console (browser session)

1. **EC2 → Instances** → select your **private** instance.  
2. Click **Connect** → tab **Session Manager** → **Start session**.  
3. A shell opens in browser (Linux: bash/ash; Windows: PowerShell).

📸 Upload later:
- `/images/3-2-3-ssm-console.png` *(Instance → Connect → Session Manager)*
- `/images/3-2-3-start-session.png` *(Start session)*
- `/images/3-2-3-shell.png` *(Interactive shell)*

> No inbound ports (22/3389) are required on the instance Security Group.

---

## B) Connect via CLI

### 1) Start an interactive shell
```bash
REGION=ap-southeast-1
INSTANCE_ID=i-0123456789abcdef0

aws ssm start-session \
  --region "$REGION" \
  --target "$INSTANCE_ID"
```

## 2) (Windows) RDP through SSM Port Forwarding
Port-forward local 13389 → 3389 on the instance, then open your RDP client to localhost:13389.

```bash
aws ssm start-session \
  --region "$REGION" \
  --target "$INSTANCE_ID" \
  --document-name AWS-StartPortForwardingSession \
  --parameters "localPortNumber=['13389'],portNumber=['3389']"
```
## 3) (Linux) SSH through SSM Port Forwarding (optional)
If you prefer your SSH client instead of the browser shell:

```bash

# Forward local 1222 → instance 22
aws ssm start-session \
  --region "$REGION" \
  --target "$INSTANCE_ID" \
  --document-name AWS-StartPortForwardingSession \
  --parameters "localPortNumber=['1222'],portNumber=['22']"

# In another terminal:
ssh -p 1222 ec2-user@127.0.0.1
```

No public IP, no inbound 22 on SG; SSH travels inside the SSM tunnel.

📸 Upload later:

/images/3-2-3-port-forward.png (Port forwarding session)

/images/3-2-3-rdp.png (RDP to localhost:13389)

## C) (Optional) Session logging
To retain audit evidence, enable Session Manager preferences to log sessions to S3 and/or CloudWatch Logs (aligns with later reporting):

/images/3-2-3-logging-prefs.png (Preferences → S3/CloudWatch destination)

**Troubleshooting**
**Session fails to start:**

Missing one of the endpoints (ssm, ssmmessages, ec2messages) or Private DNS not enabled.

Endpoint SG doesn’t allow 443 from your private subnets/VPC CIDR.

Instance role lacks AmazonSSMManagedInstanceCore or the instance can’t reach endpoints (routing/DNS).

**RDP/SSH over port-forwarding fails:**

Service not listening (RDP disabled or SSH not running), or local firewall blocks the forwarded port.

OS credentials incorrect (SSM doesn’t create users).


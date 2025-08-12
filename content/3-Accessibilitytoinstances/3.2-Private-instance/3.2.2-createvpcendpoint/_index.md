---
title : "Enable standards"
date : "`r Sys.Date()`"
weight : 2
chapter : false
pre : " <b> 3.2.2 </b> "
---


**Goal**  
Create the **VPC Interface Endpoint** for **`com.amazonaws.<region>.ec2messages`** so **SSM Agent** on private instances can exchange control messages with Systems Manager **over HTTPS (443) inside the VPC** (no internet, no bastion).

> **Prerequisites**
> - VPC has **DNS hostnames** and **DNS resolution** enabled.  
> - You already created the other two SSM endpoints:  
>   - `com.amazonaws.<region>.ssm` *(3.2.2.1)*  
>   - `com.amazonaws.<region>.ssmmessages` *(3.2.2.2)*  
> - A **Security Group** for endpoints that allows **TCP 443** **from your private subnets/VPC CIDR**.

---

## Console

1. Open **VPC → Endpoints → Create endpoint**.  
2. **Name tag**: `vpce-ec2messages`  
3. **Service category**: *AWS services*  
4. **Service name**: search and select **`com.amazonaws.<region>.ec2messages`**  
5. **VPC**: select your workshop VPC (e.g., `workshop-ps-vpc`)  
6. **Subnets**: choose **both private subnets** (e.g., `private-a`, `private-b`)  
7. **Security group**: select the **endpoint SG** that allows **HTTPS (443)** from your private CIDR(s)  
8. **Enable DNS name** / **Enable private DNS name**: **Enabled**  
9. **Policy**: *Full access* (default) or paste a restrictive policy (see below)  
10. Click **Create endpoint** and wait until **Available**.

📸 Upload later:  
- `/images/3-2-2-3-ec2messages-search.png` *(Service search: ec2messages)*  
- `/images/3-2-2-3-ec2messages-subnets.png` *(Choose private subnets + SG)*  
- `/images/3-2-2-3-ec2messages-dns.png` *(Private DNS enabled)*  
- `/images/3-2-2-3-ec2messages-ready.png` *(Endpoint status: Available)*

---

## CLI

```bash
REGION=ap-southeast-1
VPC_ID=<your-vpc-id>
PRV_A=<subnet-id-private-a>
PRV_B=<subnet-id-private-b>
SG_EP=<endpoint-sg-id>   # SG that allows 443 from your VPC CIDR

# Create ec2messages Interface Endpoint
aws ec2 create-vpc-endpoint \
  --region $REGION \
  --vpc-id $VPC_ID \
  --vpc-endpoint-type Interface \
  --service-name com.amazonaws.$REGION.ec2messages \
  --subnet-ids $PRV_A $PRV_B \
  --security-group-ids $SG_EP \
  --private-dns-enabled
```

## (Optional) Restrictive Endpoint Policy
If you need to limit who can use the endpoint, attach a policy (endpoint policy support is service-dependent; use the default unless you have a clear requirement):

json
Sao chép
Chỉnh sửa
{
  "Statement": [
    {
      "Sid": "AllowEc2MessagesFromThisAccount",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "ec2messages:*",
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "aws:PrincipalAccount": "<ACCOUNT_ID>"
        }
      }
    }
  ]
}

Apply when creating the endpoint (Console) or update later:
```bash
aws ec2 modify-vpc-endpoint-policy \
  --region $REGION \
  --vpc-endpoint-id <vpce-id> \
  --policy-document file://ec2messages-endpoint-policy.json
```

**Validate (Session Manager)**
In **EC2**, select a **private instance** with **SSM Agent** and the **AmazonSSMManagedInstanceCore** instance profile.

Click **Connect → Session Manager tab → Start session.**

If the session opens **without internet access**, your three endpoints (ssm, ssmmessages, ec2messages) are working.

If it fails, check:

**Endpoint SG allows 443 from your private CIDR.
Private DNS is enabled on the endpoint.
All three endpoints exist (ssm, ssmmessages, ec2messages).
Instance IAM role has AmazonSSMManagedInstanceCore.
VPC DNS and routing are enabled/correct.**

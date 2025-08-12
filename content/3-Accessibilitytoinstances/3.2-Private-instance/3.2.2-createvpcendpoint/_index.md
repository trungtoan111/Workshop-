| title   | date            | weight | chapter | pre              |
|---------|-----------------|--------|---------|------------------|
| HIPAA   | `r Sys.Date()`  | 2      | false   | <b> 3.2.2 </b>   |

#### Enable HIPAA Standard in Security Hub

In this section, you will **enable the HIPAA security standard** in AWS Security Hub and review **key controls** related to encryption, logging, and access governance. This ensures your environment aligns with HIPAA safeguards while you collect and triage findings centrally.

**What you’ll do**
- Enable the **HIPAA** standard in Security Hub.
- Review and (optionally) tune important controls:
  - **Encryption at rest** (S3, EBS, RDS)
  - **Encryption in transit** (TLS on ALB/ELB)
  - **Audit logging** (CloudTrail with log file validation)
  - **Access restrictions** (Block S3 public access, least privilege IAM)
- Validate that **findings** appear in Security Hub and map to remediation steps.

**Screenshots (placeholders)**
- ![Enable HIPAA Standard](/images/3.2.2-hipaa-enable.png)
- ![Review HIPAA Controls](/images/3.2.2-hipaa-controls.png)
- ![Validate HIPAA Findings](/images/3.2.2-hipaa-findings.png)

---

### Content

- **[Enable HIPAA Standard](./3.2.2.1-enable-hipaa/)**  
  Turn on the HIPAA standard in Security Hub for the target Region(s).

- **[Configure Key Controls](./3.2.2.2-configure-controls/)**  
  Focus on encryption (at rest & in transit), CloudTrail log validation, and S3 public access blocks.

- **[Validate Findings](./3.2.2.3-validate-findings/)**  
  Confirm HIPAA findings are generated and visible in Security Hub; note remediation guidance.

> Tip: If you already enabled multiple standards (SOC 2 / PCI-DSS / HIPAA), findings will aggregate automatically. Use filters (e.g., **Product name = AWS Security Hub**, **Standards = HIPAA**) to focus on HIPAA-related checks.

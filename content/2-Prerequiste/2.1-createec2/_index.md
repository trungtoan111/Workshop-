| title                                   | date            | weight | chapter | pre          |
|-----------------------------------------|-----------------|--------|---------|--------------|
| Verify AWS Config & Rules Status        | `r Sys.Date()`  | 1      | false   | <b> 2.1 </b> |

In this step, you will verify and baseline **AWS Config** so network resources are **continuously recorded**, **evaluated** against **managed rules / conformance packs**, and **auto-remediated** where possible. This is the foundation for **Network Compliance & Audit Automation**.

You will:
- Confirm **AWS Config recorder** & **delivery channel** are **enabled** in your Region.
- Ensure recording for key **network resource types**: VPC, Subnet, RouteTable, InternetGateway, NATGateway, VpcEndpoint, SecurityGroup, NetworkAcl, NetworkInterface, LoadBalancer/TargetGroup, EIP.
- Review status of **AWS Managed Rules** for networking (e.g., public SG checks, NACL, VPC flow logs, ALB/NLB HTTPS enforcement).
- (Optional) Deploy a **Conformance Pack** for network baselines and set **auto-remediation** via **SSM Automation**.
- (Optional) Configure a **Config Aggregator** (multi-account/Org) and **notifications** (EventBridge → SNS/Chat).

**What this gives you:** always-on evidence for audits, fast drift detection, and one-click fixes for common misconfigs.

{{% notice info %}}
Tip: Use **least privilege** for the Config delivery bucket/KMS, and tag all rules/packs with `Compliance=Network` to slice reports later.
{{% /notice %}}

### Content
- [Verify Config Recorder & Delivery Channel](2.1.1-verify-config-recorder/)
- [Enable Recording for Network Resource Types](2.1.2-enable-network-types/)
- [Check Managed Rules (Networking)](2.1.3-check-managed-rules/)
- [Deploy Network Conformance Pack (Optional)](2.1.4-deploy-conformance-pack/)
- [Set Auto-Remediation with SSM Automation](2.1.5-auto-remediation-ssm/)
- [Configure Config Aggregator (Optional)](2.1.6-config-aggregator/)
- [Notifications & Reporting (EventBridge/SNS)](2.1.7-notifications-reporting/)

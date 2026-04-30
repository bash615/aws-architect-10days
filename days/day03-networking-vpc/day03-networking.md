# Day 03 — VPC Design, Transit Gateway & Network Segmentation

**Course:** 10-Day AWS Platform Architect  
**Region:** eu-west-2  
**GitHub:** github.com/bash615/aws-architect-10days  
**Commit path:** `days/day03-networking/`

---

## Context from Day 02

| Component | Value |
|---|---|
| Management Account | 163209926128 |
| Organisation ID | o-ghty80xw82 |
| Control Tower Region | eu-west-2 (39 guardrails) |
| CloudTrail Admin Account | 363918845564 |
| Config Aggregator Account | 523835807624 |
| GuardDuty Delegated Admin | 523835807624 |
| IAM Identity Center | Enabled |
| Region Deny SCP | Active — eu-west-2 only |

---

## Session Objectives

- Design a hub-and-spoke VPC architecture using Transit Gateway
- Build segmented VPCs: Shared Services, Production, Non-Production, Inspection
- Implement network segmentation with TGW route tables
- Deploy VPC Flow Logs → S3 / CloudWatch
- Configure DNS resolution with Route 53 Resolver
- Enforce network guardrails via SCPs and AWS Network Firewall
- Commit all IaC to GitHub `days/day03-networking/`

---

## 1. Architecture Overview

### Hub-and-Spoke

Transit Gateway is the hub. All spoke VPCs connect via TGW attachments.  
East-west traffic is forced through an Inspection VPC running AWS Network Firewall.  
No VPC-to-VPC peering, ever.

```
[Prod VPC] ──┐
             ├──→ TGW → [Inspection VPC (Network Firewall)] → Shared Services / Internet
[Non-Prod] ──┘
```

### CIDR Plan

| VPC | CIDR | Account | Purpose |
|---|---|---|---|
| Shared Services | 10.0.0.0/22 | Shared Svcs (new) | DNS, AD, patch, tooling |
| Inspection | 10.0.4.0/24 | Shared Svcs | AWS Network Firewall, egress |
| Production | 10.1.0.0/16 | Prod (new) | Workloads — isolated |
| Non-Production | 10.2.0.0/16 | Non-Prod (new) | Dev/test workloads |
| Management | 10.0.8.0/22 | 163209926128 | Bastion, control plane |

### TGW Route Table Design

| TGW Route Table | Associated Attachments | Propagations | Static Routes |
|---|---|---|---|
| pre-inspection-rt | Prod VPC, Non-Prod VPC | None | 0.0.0.0/0 → Inspection TGW attach |
| post-inspection-rt | Inspection VPC (egress ENI) | Shared Svcs VPC | Specific CIDRs to spokes |
| shared-services-rt | Shared Svcs VPC | All spoke VPCs | 0.0.0.0/0 → Inspection |

---

## 2. Step-by-Step Build

### Step 1 — Create OU and Accounts

- Create **Network OU** under Root in AWS Organizations (Management Account)
- Vend **Shared Services** account via Account Factory in Control Tower
- Vend **Production** account via Account Factory
- Vend **Non-Production** account via Account Factory

> Account Factory auto-applies SCPs from parent OU. Verify eu-west-2 deny SCP is inherited.

```bash
# IAM Identity Center Permission Set for networking role
NetworkAdmin: AdministratorAccess (scoped to Network OU accounts)
```

---

### Step 2 — Create Transit Gateway

Deploy TGW in the Shared Services account. RAM-share to all spoke accounts.

```bash
aws ec2 create-transit-gateway \
  --description 'Org TGW eu-west-2' \
  --options AmazonSideAsn=64512,AutoAcceptSharedAttachments=disable,\
DefaultRouteTableAssociation=disable,DefaultRouteTablePropagation=disable,\
DnsSupport=enable,VpnEcmpSupport=enable \
  --tag-specifications 'ResourceType=transit-gateway,Tags=[{Key=Name,Value=org-tgw-euw2}]'
```

- RAM share TGW to all accounts in the Organisation
- `DefaultRouteTableAssociation=disable` — manage RT assignment manually
- Note TGW ID: `tgw-xxxxxxxxxxxxxxxxx`

---

### Step 3 — Build VPCs and Subnets

Each VPC follows a consistent 3-tier subnet layout:

| Tier | Purpose | Route Table | NACLs |
|---|---|---|---|
| `/public-*` | ALB, NAT GW (Shared Svcs only) | IGW default route | Restrictive inbound |
| `/app-*` | Compute, EKS, Lambda | TGW attachment route | Deny direct internet |
| `/data-*` | RDS, ElastiCache | TGW only | DB ports only from app tier |
| `/tgw-*` | TGW ENI attachment subnets | Local only | No direct traffic |

> ⚠️ TGW attachment subnets must be /28 minimum. Use dedicated subnets — do not share with workloads.

---

### Step 4 — Create TGW Attachments and Route Tables

- Create VPC attachment in each spoke account (accept in TGW owner account)
- Create three TGW Route Tables: `pre-inspection-rt`, `post-inspection-rt`, `shared-services-rt`
- Associate each attachment with the correct route table
- Enable route propagation where required
- Add static routes: all spoke default routes point to Inspection VPC attachment

```bash
# Associate Prod VPC attachment to pre-inspection-rt
aws ec2 associate-transit-gateway-route-table \
  --transit-gateway-route-table-id tgw-rtb-XXXXX \
  --transit-gateway-attachment-id tgw-attach-XXXXX
```

---

### Step 5 — Deploy AWS Network Firewall (Inspection VPC)

- Create Network Firewall in Inspection VPC
- Create Firewall Policy with stateless default: forward to stateful
- Stateful rule groups: domain allowlist (Suricata IPS compatible)
- Update Inspection VPC route tables: traffic in from TGW → Firewall, Firewall → TGW

> Network Firewall endpoints are AZ-specific. Deploy one per AZ for HA.

---

### Step 6 — VPC Flow Logs

```bash
# Enable flow logs on each VPC — ship to S3 in Security account (523835807624)
aws ec2 create-flow-logs \
  --resource-type VPC \
  --resource-ids vpc-XXXXX \
  --traffic-type ALL \
  --log-destination-type s3 \
  --log-destination arn:aws:s3:::org-flowlogs-523835807624-euw2 \
  --log-format '${version} ${account-id} ${vpc-id} ${srcaddr} ${dstaddr} \
  ${srcport} ${dstport} ${protocol} ${packets} ${bytes} ${action} ${log-status}'
```

---

### Step 7 — Route 53 Resolver

- Create **outbound** Resolver endpoint in Shared Services VPC (for on-prem forwarding)
- Create **inbound** Resolver endpoint (for on-prem DNS to resolve AWS records)
- Create Resolver Rules: forward `.internal.example.com` → on-prem DNS IPs
- Share Resolver Rules via RAM to all accounts in the org

> Spoke VPCs use VPC DHCP option sets pointing to Shared Services Resolver endpoint IPs.

---

## 3. Network Guardrails — SCPs

### Deny VPC Peering

```json
{
  "Sid": "DenyVPCPeering",
  "Effect": "Deny",
  "Action": [
    "ec2:CreateVpcPeeringConnection",
    "ec2:AcceptVpcPeeringConnection"
  ],
  "Resource": "*"
}
```

### Deny Internet Gateway in Spoke Accounts

```json
{
  "Sid": "DenyIGWCreation",
  "Effect": "Deny",
  "Action": [
    "ec2:AttachInternetGateway",
    "ec2:CreateInternetGateway"
  ],
  "Resource": "*",
  "Condition": {
    "ArnNotLike": {
      "aws:PrincipalArn": "arn:aws:iam::*:role/NetworkAdmin"
    }
  }
}
```

### Deny NAT Gateway in Spoke Accounts

```json
{
  "Sid": "DenyNATGatewayInSpokes",
  "Effect": "Deny",
  "Action": "ec2:CreateNatGateway",
  "Resource": "*"
}
```

> ⚠️ Apply DenyNATGateway and DenyIGW SCPs to Prod and Non-Prod OUs only. Shared Services needs NAT GW for centralised egress.

---

## 4. IaC Structure — Terraform

| Module | Path | Resources |
|---|---|---|
| transit-gateway | `modules/tgw/` | TGW, attachments, route tables, RAM share |
| vpc-shared-services | `modules/vpc-shared-svcs/` | VPC, subnets, NACLs, resolver endpoints |
| vpc-inspection | `modules/vpc-inspection/` | VPC, Network Firewall, subnets |
| vpc-workload | `modules/vpc-workload/` | Generic workload VPC, TGW attach |
| flow-logs | `modules/flow-logs/` | Flow logs, S3 bucket policy, KMS |
| network-scps | `modules/network-scps/` | SCP documents + policy attachments |

```
days/day03-networking/
├── main.tf
├── variables.tf
├── outputs.tf
├── backend.tf          # S3 backend in Management account
├── providers.tf        # assume-role providers per account
└── modules/
    ├── tgw/
    ├── vpc-shared-svcs/
    ├── vpc-inspection/
    ├── vpc-workload/
    ├── flow-logs/
    └── network-scps/
```

---

## 5. Validation Checklist

| Test | Method | Expected Result | ✓ |
|---|---|---|---|
| Prod → Shared Services | ping / traceroute from EC2 | Reaches via Inspection VPC (3 hops) | |
| Prod → Non-Prod (blocked) | Security Group + TGW RT | No route — traffic dropped | |
| Shared Svcs → All spokes | ping from resolver endpoint | Reachable | |
| Internet egress | curl ifconfig.me from Prod EC2 | NAT GW IP of Inspection VPC | |
| VPC Flow Logs | Generate traffic; check S3 bucket | Logs appear within 10 mins | |
| DNS resolution | nslookup internal.example.com | Resolves via Shared Svcs endpoint | |
| SCP: deny peering | Attempt CreateVpcPeeringConnection | Access Denied | |
| SCP: deny IGW | Attempt AttachInternetGateway | Access Denied | |
| Network Firewall | curl blocked domain from Prod | Connection refused / DROP logged | |
| GuardDuty findings | Check Security account dashboard | No unexpected findings | |

---

## 6. Key Design Decisions

| Decision | Value |
|---|---|
| TGW ASN | |
| Inspection VPC model (centralised/distributed) | |
| Network Firewall HA strategy | |
| Flow logs destination (S3 / CWL / both) | |
| DNS forward zones | |
| NAT Gateway count and AZ spread | |
| Shared Services account ID | |
| Production account ID | |
| Non-Production account ID | |

---

## 7. Common Issues

| Symptom | Likely Cause | Fix |
|---|---|---|
| TGW attachment stuck pending-acceptance | Must accept in TGW owner account | `ec2 accept-transit-gateway-vpc-attachment` |
| Route not propagating | RT propagation not enabled for attachment | Enable propagation on correct RT |
| Flow logs not appearing | IAM role missing s3:PutObject to Security acct bucket | Check bucket policy + cross-account trust |
| DNS resolution failing in spoke | DHCP option set not updated | Create DHCP option set pointing to Shared Svcs resolver endpoint IPs |
| Network Firewall drops all traffic | Route table order wrong in Inspection VPC | Traffic must hit firewall ENI before returning to TGW |
| SCP blocks NetworkAdmin role | Condition not excluding NetworkAdmin ARN | Add ArnNotLike condition on principal ARN |

---

## 8. Day 03 Completion Checklist

- [ ] TGW deployed, RAM-shared to all accounts
- [ ] Shared Services VPC built and attached to TGW
- [ ] Inspection VPC with Network Firewall deployed
- [ ] Production VPC attached to TGW (pre-inspection RT)
- [ ] Non-Production VPC attached to TGW (pre-inspection RT)
- [ ] TGW route tables created and all attachments associated
- [ ] East-west traffic validated through Inspection VPC
- [ ] VPC Flow Logs shipping to Security account S3 bucket
- [ ] Route 53 Resolver endpoints deployed and rules RAM-shared
- [ ] Network SCPs applied to Prod/Non-Prod OUs
- [ ] All Terraform committed to `days/day03-networking/`
- [ ] README.md updated with account IDs and CIDR map

---

## Looking Ahead — Day 04

Day 04: Security Controls — Security Hub, Config Rules, Macie, Inspector, centralised logging pipeline.

- Security Hub delegated admin → 523835807624
- Config rules: required-tags, encrypted-volumes, rds-storage-encrypted
- Macie enabled for S3 data classification
- CloudTrail → CloudWatch Logs → EventBridge alerting
- Inspector v2 activated org-wide

---

*Day 03 of 10 — AWS Platform Architect*

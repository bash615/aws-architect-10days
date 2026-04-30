# AWS Platform Architect — 10-Day Mastery Tracker
> **Profile**: 25+ years cloud/virtualisation | Kubernetes lab with monitoring active | Zero AWS spend target
> **Strategy**: AWS for architecture/IAM/networking/security concepts — heavy workloads stay on K8s lab

---

## Architecture Philosophy
```
K8s Lab (Your workloads)     ←→     AWS Free Tier (Architecture, IAM, Networking, Security)
   Prometheus/Grafana                  CloudWatch concepts (no cost)
   Running containers                  ECR/ECS/EKS concepts (design only)
   CI/CD pipelines                     CodePipeline/CodeBuild (design patterns)
   Helm charts                         CloudFormation/Terraform patterns
```

---

## GitHub Repository Structure
```
aws-architect-10days/
├── README.md
├── .gitignore
├── days/
│   ├── day01-foundation-iam-billing/
│   │   ├── README.md
│   │   ├── notes.md
│   │   └── diagrams/
│   ├── day02-organisations-control-tower/
│   ├── day03-networking-vpc/
│   ├── day04-compute-containers/
│   ├── day05-storage-databases/
│   ├── day06-security-compliance/
│   ├── day07-monitoring-observability/
│   ├── day08-cicd-devops/
│   ├── day09-serverless-messaging/
│   └── day10-architecture-review/
├── terraform/
│   ├── modules/
│   └── environments/
├── k8s-aws-integration/
│   ├── irsa/
│   ├── external-secrets/
│   └── crossplane/
└── diagrams/
    └── architecture/
```

---

## Git Setup Commands
```bash
# Initial setup
git init aws-architect-10days
cd aws-architect-10days
git remote add origin https://github.com/YOUR_USERNAME/aws-architect-10days.git

# Daily workflow
git add .
git commit -m "Day X: [topic] - [what was done]"
git push origin main

# Branch per day (recommended)
git checkout -b day-01-foundation
# ... do work ...
git checkout main && git merge day-01-foundation
git push origin main
```

---

## Daily Progress Tracker

| Day | Topic | Status | Hours | Git Commit | Notes |
|-----|-------|--------|-------|------------|-------|
| 01 | Foundation, IAM, Billing Safety | 🟡 In Progress | 0/10 | - | |
| 02 | AWS Organisations, Control Tower, Landing Zone | ⬜ Pending | 0/10 | - | |
| 03 | VPC, Networking, Security Groups, NACLs | ⬜ Pending | 0/10 | - | |
| 04 | Compute: EC2 concepts, ECS, EKS (K8s integration) | ⬜ Pending | 0/10 | - | |
| 05 | Storage: S3, EBS, EFS + RDS/Aurora concepts | ⬜ Pending | 0/10 | - | |
| 06 | Security: KMS, Secrets Manager, GuardDuty, SCPs | ⬜ Pending | 0/10 | - | |
| 07 | Monitoring: CloudWatch, X-Ray, K8s integration | ⬜ Pending | 0/10 | - | |
| 08 | CI/CD: CodePipeline, CodeBuild, GitOps patterns | ⬜ Pending | 0/10 | - | |
| 09 | Serverless: Lambda, SQS, SNS, EventBridge | ⬜ Pending | 0/10 | - | |
| 10 | Architecture Review, Well-Architected, Mock Interview | ⬜ Pending | 0/10 | - | |

**Legend**: ✅ Complete | 🟡 In Progress | ⬜ Pending | ❌ Blocked

---

## Day 01 — AWS Foundation, IAM and Billing Safety
**Date**: ___________  
**Hours target**: 10  
**Hours logged**: ___

### Learning Objectives
- [ ] Understand AWS account structure and security boundaries
- [ ] Know the difference between root, IAM users, roles and federated access
- [ ] Apply least privilege thinking to IAM design
- [ ] Understand IAM Identity Center for enterprise access
- [ ] Set billing guardrails before doing anything else
- [ ] Design for multi-account architecture from day one

### AWS Free Account Setup
```
URL: https://aws.amazon.com/free
Required:
  - Email address (use a dedicated one, e.g. aws-lab@yourdomain.com)
  - Phone number (for verification)
  - Credit/debit card (required but NOT charged under free tier)
  - Address

Free Tier limits (important):
  - 750 hrs/month EC2 t2.micro or t3.micro (12 months)
  - 5 GB S3 storage (12 months)
  - 25 GB DynamoDB (always free)
  - 1M Lambda invocations/month (always free)
  - 30 GB EBS storage (12 months)
```

### Root Account Hardening Checklist
- [ ] Sign in as root
- [ ] Enable MFA on root (use Authenticator app — Google/Authy/1Password)
- [ ] Never create access keys for root
- [ ] Store root credentials in a password manager
- [ ] Set alternate contact (billing, operations, security)
- [ ] Enable IAM access to billing console

### Region Selection
```
Primary region: eu-west-2 (London)
Reason: Closest to you, GDPR-compliant, full service availability
```

### IAM Tasks
- [ ] Go to IAM → Dashboard — review security status
- [ ] Create an IAM admin user (do NOT use root for daily work)
- [ ] Create IAM group: `Administrators`
- [ ] Attach policy: `AdministratorAccess` to group
- [ ] Add your admin user to the group
- [ ] Enable MFA on admin user
- [ ] Create access keys ONLY if needed for CLI (rotate regularly)

### IAM Identity Center
- [ ] Check if IAM Identity Center is enabled
- [ ] Understand: this replaces IAM users in enterprise
- [ ] Note: For multi-account → IAM Identity Center + permission sets
- [ ] This is the AWS SSO replacement — used in every enterprise AWS shop

### Billing Budget Alert
```
Billing → Budgets → Create Budget
  Name: aws-lab-budget-10days
  Amount: £5 GBP
  Alert: 50% (£2.50) and 80% (£4.00)
  Email: your-email@domain.com
```
- [ ] Budget created
- [ ] Test alert email received

### Free Tier Monitoring
- [ ] Billing → Free Tier → Review all services
- [ ] Enable billing alerts in CloudWatch (Billing → Billing Preferences)
- [ ] Note current $0.00 spend

### K8s Lab ↔ AWS Concepts Mapping (Day 01)

| Your K8s Lab | AWS Equivalent | Notes |
|-------------|----------------|-------|
| RBAC ClusterRole/Role | IAM Role + Policy | Both are least-privilege identity |
| ServiceAccount | IAM Role for Service Account (IRSA) | K8s SA → AWS IAM Role via OIDC |
| Namespace isolation | AWS Account isolation | Accounts = hard boundary |
| kubeconfig | AWS CLI credentials / assume-role | Both grant temporary access |
| Prometheus RBAC | CloudWatch permissions | Same concept, different plane |

### Architecture Decision Records (ADR)
```markdown
# ADR-001: Primary Region
Decision: eu-west-2 (London)
Reason: Latency, GDPR, full service availability
Date: [today]

# ADR-002: Root Account Usage
Decision: Root account locked down, MFA enforced, never used for daily work
Reason: Blast radius reduction, security best practice
Date: [today]

# ADR-003: IAM Strategy
Decision: IAM roles preferred over IAM users; IAM Identity Center for workforce
Reason: Temporary credentials, no long-term secrets, enterprise scale
Date: [today]
```

### Interview Talking Points (Day 01)
> "I treat AWS account setup as an architecture decision. Day one is about securing the root account with MFA, establishing least-privilege IAM, implementing IAM Identity Center for centralised access governance, locking down billing with budget alerts, and choosing a primary region. Only then do I deploy workloads."

### CV Line
> Designed AWS account foundation patterns: MFA enforcement, IAM least-privilege, IAM Identity Center, billing guardrails, budget alerts, region governance.

### Day 01 Completion Checklist
- [ ] AWS free account created and verified
- [ ] Root MFA enabled
- [ ] IAM admin user created with MFA
- [ ] Billing budget alert set (£5 max)
- [ ] Free Tier monitoring reviewed
- [ ] IAM Identity Center reviewed
- [ ] Primary region confirmed: eu-west-2
- [ ] Day 01 notes committed to GitHub
- [ ] ADRs documented

---

## Day 02 — AWS Organisations, Control Tower, Landing Zone
**Date**: ___________  
**Hours target**: 10

### Topics
- AWS Organizations: master account, member accounts, OUs
- Service Control Policies (SCPs) — the hard boundary layer
- AWS Control Tower — multi-account factory
- Landing Zone concept — account vending machine
- Security account, Log archive account pattern
- Dev/Test/Prod account separation
- K8s lab integration: treat your K8s cluster as a "workload account"

### Key Concepts
```
Root OU
├── Security OU
│   ├── Security Account (GuardDuty master, Security Hub)
│   └── Log Archive Account (CloudTrail, Config)
├── Infrastructure OU
│   └── Shared Services Account (VPC, DNS, ECR)
├── Workloads OU
│   ├── Dev Account
│   ├── Test Account
│   └── Prod Account
└── Sandbox OU
    └── Your Lab Account (free tier)
```

### SCP Examples to Design
```json
// Deny all regions except eu-west-2
{
  "Effect": "Deny",
  "Action": "*",
  "Resource": "*",
  "Condition": {
    "StringNotEquals": {
      "aws:RequestedRegion": ["eu-west-2", "us-east-1"]
    }
  }
}
```

### Checklist
- [ ] Understand Organizations hierarchy
- [ ] Design OU structure (paper/diagram)
- [ ] Understand Control Tower account factory
- [ ] Draft 3 SCP policies (region lock, service deny, tag enforcement)
- [ ] Map your K8s lab to sandbox account pattern
- [ ] Commit architecture diagram to GitHub

---

## Day 03 — VPC, Networking, Security Groups, NACLs
**Date**: ___________  
**Hours target**: 10

### Topics
- VPC design: CIDR planning, subnets, availability zones
- Public vs private subnets
- Internet Gateway vs NAT Gateway (cost trap!)
- Security Groups (stateful) vs NACLs (stateless)
- VPC Peering, Transit Gateway, PrivateLink
- Route tables
- K8s networking comparison: CNI, pod CIDR, network policies

### CIDR Planning Template
```
VPC: 10.0.0.0/16

Public Subnets (one per AZ):
  10.0.1.0/24  — eu-west-2a (public)
  10.0.2.0/24  — eu-west-2b (public)
  10.0.3.0/24  — eu-west-2c (public)

Private Subnets (one per AZ):
  10.0.11.0/24 — eu-west-2a (private)
  10.0.12.0/24 — eu-west-2b (private)
  10.0.13.0/24 — eu-west-2c (private)

Data Subnets (RDS, ElastiCache):
  10.0.21.0/24 — eu-west-2a (data)
  10.0.22.0/24 — eu-west-2b (data)
  10.0.23.0/24 — eu-west-2c (data)
```

### K8s ↔ AWS Networking Mapping

| K8s Concept | AWS Equivalent |
|-------------|----------------|
| Pod CIDR | VPC CIDR |
| CNI plugin | VPC networking |
| NetworkPolicy | Security Group + NACL |
| ClusterIP service | Internal Load Balancer |
| NodePort/LoadBalancer | ALB/NLB |
| Ingress | Application Load Balancer |
| Namespace network isolation | Security Group segregation |

### Checklist
- [ ] VPC created (free, no cost)
- [ ] Subnets designed across 3 AZs
- [ ] Security Group rules designed for K8s workloads
- [ ] Internet Gateway attached (no cost)
- [ ] Route tables configured
- [ ] NAT Gateway: designed but NOT deployed (cost item)
- [ ] Diagram committed to GitHub

---

## Day 04 — Compute: EC2, ECS, EKS and K8s Integration
**Date**: ___________  
**Hours target**: 10

### Topics
- EC2: instance types, AMIs, launch templates, Auto Scaling
- ECS: task definitions, services, Fargate vs EC2 mode
- EKS: managed Kubernetes — compare to your K8s lab
- IRSA: IAM Roles for Service Accounts (huge topic)
- ECR: private container registry
- K8s workloads on your lab vs EKS architecture

### IRSA — Critical Concept (K8s + AWS)
```
Your K8s Lab Pattern:
  Pod → ServiceAccount → kube-apiserver → RBAC

AWS EKS IRSA Pattern:
  Pod → ServiceAccount (annotated) → OIDC Provider → IAM Role → AWS API

# The annotation:
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-app
  namespace: default
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::ACCOUNT_ID:role/my-app-role
```

### Checklist
- [ ] Understand EC2 free tier (t2.micro/t3.micro — 750hrs/month)
- [ ] Compare ECS Fargate vs your K8s deployments
- [ ] Design IRSA pattern for K8s-to-AWS access
- [ ] Review ECR (free tier: 500MB private, unlimited public)
- [ ] Design EKS architecture (do NOT deploy — cost item)
- [ ] Map your Helm charts to ECS task definitions conceptually
- [ ] Commit IRSA example manifests to GitHub

---

## Day 05 — Storage: S3, EBS, EFS + Databases
**Date**: ___________  
**Hours target**: 10

### Topics
- S3: buckets, versioning, lifecycle, encryption, bucket policies
- EBS: volumes, snapshots, types (gp3 vs io2)
- EFS: shared storage, K8s PersistentVolume integration
- RDS/Aurora: managed databases
- DynamoDB: serverless NoSQL
- Secrets Manager vs SSM Parameter Store

### K8s Storage Integration
```yaml
# EFS as K8s PersistentVolume (real integration)
apiVersion: v1
kind: PersistentVolume
metadata:
  name: efs-pv
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  csi:
    driver: efs.csi.aws.com
    volumeHandle: fs-0123456789abcdef0
```

### Checklist
- [ ] S3 bucket created (free tier: 5GB)
- [ ] Enable versioning on lab bucket
- [ ] Test S3 bucket policy (deny unencrypted uploads)
- [ ] Understand EFS for K8s ReadWriteMany workloads
- [ ] DynamoDB table created (always free tier)
- [ ] Secrets Manager vs SSM Parameter Store comparison
- [ ] Commit storage patterns to GitHub

---

## Day 06 — Security: KMS, GuardDuty, Security Hub, SCPs
**Date**: ___________  
**Hours target**: 10

### Topics
- AWS KMS: key management, envelope encryption
- GuardDuty: threat detection
- Security Hub: centralised findings
- AWS Config: compliance as code
- CloudTrail: audit logging (always on)
- Macie: S3 data classification
- Detective: investigation
- IAM Access Analyzer

### Security Boundary Design for K8s
```
External Traffic
    ↓
AWS WAF (L7 firewall)
    ↓
ALB (with HTTPS termination)
    ↓
Security Group (allow 443 only)
    ↓
[Your K8s Ingress / Node]
    ↓
K8s NetworkPolicy
    ↓
Pod (your container)
    ↓
ServiceAccount + IRSA (AWS API calls)
    ↓
KMS encrypted secrets (from Secrets Manager)
```

### Checklist
- [ ] CloudTrail enabled (free: management events)
- [ ] GuardDuty: understand findings (30-day free trial)
- [ ] IAM Access Analyzer: review (free)
- [ ] Design KMS key policy for K8s secrets
- [ ] Design Security Hub integration pattern
- [ ] Map K8s security controls to AWS equivalents
- [ ] Commit security architecture to GitHub

---

## Day 07 — Monitoring: CloudWatch, X-Ray, K8s Integration
**Date**: ___________  
**Hours target**: 10

### Topics
- CloudWatch: metrics, logs, alarms, dashboards
- CloudWatch Container Insights for K8s
- X-Ray: distributed tracing
- Prometheus + CloudWatch integration
- AWS Managed Prometheus (AMP)
- AWS Managed Grafana (AMG)
- EventBridge: event-driven operations

### Your K8s Lab → AWS Monitoring Bridge
```
Your Lab:                          AWS:
Prometheus metrics    →    CloudWatch custom metrics (via CW agent)
Grafana dashboards    →    Amazon Managed Grafana
Alertmanager          →    CloudWatch Alarms + SNS
Jaeger/tracing        →    AWS X-Ray
Loki logs             →    CloudWatch Logs
```

### Checklist
- [ ] Understand CloudWatch metrics namespaces
- [ ] Design K8s → CloudWatch metrics forwarding
- [ ] Review Container Insights for EKS
- [ ] AWS Managed Prometheus: architecture design
- [ ] Create one CloudWatch Alarm (free tier: 10 alarms)
- [ ] Design cross-platform observability strategy
- [ ] Commit monitoring architecture to GitHub

---

## Day 08 — CI/CD: CodePipeline, CodeBuild, GitOps
**Date**: ___________  
**Hours target**: 10

### Topics
- CodeCommit (or GitHub Actions with OIDC to AWS)
- CodeBuild: container-based build service
- CodePipeline: orchestration
- ECR: container registry
- GitHub Actions + AWS OIDC (no long-term credentials!)
- GitOps: ArgoCD on your K8s lab → AWS ECR
- Terraform: IaC for AWS (free!)

### GitHub Actions → AWS via OIDC (No Access Keys!)
```yaml
# .github/workflows/deploy.yml
jobs:
  deploy:
    permissions:
      id-token: write
      contents: read
    steps:
      - uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::ACCOUNT:role/github-actions-role
          aws-region: eu-west-2
```

### K8s Lab CI/CD Integration
```
GitHub repo
    ↓
GitHub Actions (build + test)
    ↓
AWS OIDC (no credentials)
    ↓
ECR push (container image)
    ↓
ArgoCD on your K8s lab (GitOps pull)
    ↓
Deployment to your K8s cluster
```

### Checklist
- [ ] Design GitHub Actions → AWS OIDC trust
- [ ] Create IAM role for GitHub Actions (limited permissions)
- [ ] ECR repo created (free tier)
- [ ] Push a test image to ECR
- [ ] Design GitOps pipeline with ArgoCD
- [ ] Terraform: init, plan on a simple resource
- [ ] Commit CI/CD pipeline code to GitHub

---

## Day 09 — Serverless: Lambda, SQS, SNS, EventBridge, API Gateway
**Date**: ___________  
**Hours target**: 10

### Topics
- Lambda: functions, triggers, layers, cold starts
- SQS: queues, DLQ, visibility timeout
- SNS: topics, subscriptions, fan-out patterns
- EventBridge: event bus, rules, targets
- API Gateway: REST vs HTTP vs WebSocket
- Step Functions: workflow orchestration
- Integration with K8s: event-driven workloads

### Event-Driven Architecture Pattern
```
External Event → EventBridge → SQS → Lambda → K8s Job (via kubectl/API)
                     ↓
                  SNS Topic
                     ↓
            Email / Slack / PagerDuty
```

### Checklist
- [ ] Lambda function created (Python/Node — free tier: 1M invocations)
- [ ] SQS queue created (free tier: 1M requests)
- [ ] SNS topic with email subscription
- [ ] EventBridge rule: trigger Lambda on schedule
- [ ] API Gateway → Lambda integration
- [ ] Design K8s Job triggered by SQS message
- [ ] Commit serverless patterns to GitHub

---

## Day 10 — Architecture Review, Well-Architected, Mock Interview
**Date**: ___________  
**Hours target**: 10

### Topics
- AWS Well-Architected Framework: 6 pillars
- Architecture review process
- Cost optimisation strategies
- The full 10-day architecture summary
- Mock architect interview questions
- CV and portfolio finalisation

### Well-Architected 6 Pillars
| Pillar | Key Questions |
|--------|---------------|
| Operational Excellence | How do you manage operations? IaC, observability, runbooks |
| Security | Least privilege, encryption at rest/transit, defence in depth |
| Reliability | Multi-AZ, auto-scaling, backup/restore, chaos engineering |
| Performance Efficiency | Right-sizing, managed services, serverless where appropriate |
| Cost Optimisation | Reserved instances, spot, rightsizing, K8s for workloads |
| Sustainability | Reduce footprint, use managed services, right-size |

### Final Architecture: 10-Day Lab Architecture
```
┌─────────────────────────────────────────────────────┐
│                  Your K8s Lab                        │
│  ┌──────────┐  ┌──────────┐  ┌──────────────────┐  │
│  │ Workloads│  │Prometheus│  │    ArgoCD/GitOps  │  │
│  │(containers│  │ /Grafana │  │                  │  │
│  └────┬─────┘  └────┬─────┘  └────────┬─────────┘  │
└───────┼─────────────┼─────────────────┼─────────────┘
        │             │                 │
        ▼             ▼                 ▼
┌────────────────────────────────────────────────────┐
│                   AWS (Free Tier)                   │
│  IAM/IRSA    CloudWatch    ECR        EventBridge   │
│  S3          Secrets Mgr  CodeBuild  Lambda         │
│  VPC Design  GuardDuty    CloudTrail  SQS/SNS       │
└────────────────────────────────────────────────────┘
```

### Mock Interview Questions
1. "Walk me through how you'd design a secure multi-account AWS landing zone."
2. "How do you integrate Kubernetes workloads with AWS IAM securely?"
3. "What's the difference between a Security Group and a NACL, and when do you use each?"
4. "How would you implement zero-trust networking between your K8s cluster and AWS services?"
5. "Describe your approach to cost governance in AWS."
6. "How does IRSA work and why is it preferred over node-level IAM roles?"
7. "Walk me through your AWS Well-Architected review process."

### Checklist
- [ ] Well-Architected review completed for lab architecture
- [ ] Full architecture diagram finalised
- [ ] All 10-day GitHub commits reviewed and cleaned up
- [ ] README.md updated with full learning summary
- [ ] Mock interview questions practised
- [ ] CV updated with AWS architect competencies

---

## Cost Control — Running Total

| Resource | Free Tier Limit | Cost if Exceeded | Status |
|----------|----------------|------------------|--------|
| EC2 t2/t3.micro | 750 hrs/month | ~$0.013/hr | ⬜ Not started |
| S3 | 5 GB storage | $0.023/GB/month | ⬜ Not started |
| Lambda | 1M invocations | $0.20/1M | ⬜ Not started |
| DynamoDB | 25 GB | $0.25/GB | ⬜ Not started |
| CloudWatch | 10 metrics, 5GB logs | $0.30/metric | ⬜ Not started |
| Data Transfer OUT | 1 GB/month | $0.09/GB | ⬜ Not started |
| **AVOID** | NAT Gateway | **$0.045/hr + data** | ❌ Never deploy |
| **AVOID** | EKS cluster | **$0.10/hr** | ❌ Use your K8s lab |
| **AVOID** | RDS (beyond free) | **$0.02/hr+** | ❌ DynamoDB only |

**Budget alert set at £5 — you should spend £0**

---

## Key AWS CLI Commands (Daily Use)
```bash
# Verify identity
aws sts get-caller-identity

# Configure CLI
aws configure --profile lab
# Enter: Access Key ID, Secret, eu-west-2, json

# List S3 buckets
aws s3 ls

# List EC2 instances
aws ec2 describe-instances --region eu-west-2

# Assume a role
aws sts assume-role --role-arn arn:aws:iam::ACCOUNT:role/ROLE --role-session-name lab

# Check IAM permissions
aws iam simulate-principal-policy \
  --policy-source-arn arn:aws:iam::ACCOUNT:user/USERNAME \
  --action-names s3:GetObject
```

---

## Completion Certificate Checklist
- [ ] Day 01: Foundation, IAM, Billing ✅
- [ ] Day 02: Organisations, Control Tower ✅
- [ ] Day 03: VPC, Networking ✅
- [ ] Day 04: Compute, EKS/K8s Integration ✅
- [ ] Day 05: Storage, Databases ✅
- [ ] Day 06: Security, Compliance ✅
- [ ] Day 07: Monitoring, Observability ✅
- [ ] Day 08: CI/CD, GitOps ✅
- [ ] Day 09: Serverless, Messaging ✅
- [ ] Day 10: Architecture Review, Interview Prep ✅
- [ ] GitHub repo complete with all daily commits ✅
- [ ] Architecture diagrams committed ✅
- [ ] £0 AWS spend achieved ✅

---

*Last updated: Day 01 | Tracker version: 1.0*

---

## Day 01 Completion Log
Date: 2026-04-30

### Completed Tasks
- [x] Logged into AWS Console (Account: Bash, ID: 163209926128)
- [x] Region confirmed: eu-west-2 (London)
- [x] Root MFA verified (Passkey/Security key)
- [x] Terminated rogue kubeadm-node-1 EC2 instance (m7i.flex.large) - stopped $3.48/month charge
- [x] Zero spend budget alert created (alerts at $0.01)
- [x] IAM Administrators group created with AdministratorAccess policy
- [x] IAM admin user bash-admin created and added to Administrators group
- [x] No root access keys confirmed

### Architecture Decisions
- Root account: locked down, MFA only, never used for daily work
- Daily work: use bash-admin IAM user via https://163209926128.signin.aws.amazon.com/console
- Primary region: eu-west-2 (London)
- Billing: zero spend budget, credits cover any accidental usage

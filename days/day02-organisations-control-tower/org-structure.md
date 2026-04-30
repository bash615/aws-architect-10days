# AWS Organisation Structure — As Built
## Day 02 — AWS Platform Architect Course

---

## Account Inventory

| Account Name         | Account ID   | Role                        | OU        |
|----------------------|--------------|-----------------------------|-----------|
| Bash (Management)    | 163209926128 | Management Account          | Root      |
| Config Aggregator    | 523835807624 | AWS Config Aggregator       | Security  |
| CloudTrail Admin     | 363918845564 | CloudTrail Administrator    | Security  |

---

## OU Tree

```
Root
├── Management Account (163209926128) — Bash
│   └── [No workloads — management only]
│
├── Security OU  [Foundational]
│   ├── Config Aggregator (523835807624)
│   └── CloudTrail Administrator (363918845564)
│
└── Sandbox OU  [Recommended]
    └── [Future: dev/test member accounts]
```

---

## Planned Future OU Structure

```
Root
├── Security OU          — Log archive, audit, Config aggregator
├── Infrastructure OU    — Network hub, Transit Gateway, shared services
├── Workloads OU
│   ├── Dev              — Non-prod accounts, looser guardrails
│   └── Prod             — Strict SCPs, no public S3, no root activity
├── Sandbox OU           — Engineer sandboxes, 30-day expiry
└── Suspended OU         — Offboarded accounts awaiting closure
```

---

## Service Integrations Enabled

| Service              | Status   | Account              |
|----------------------|----------|----------------------|
| AWS Organizations    | Enabled  | 163209926128         |
| AWS Control Tower    | Enabled  | eu-west-2 (London)   |
| AWS Config           | Enabled  | 523835807624         |
| AWS CloudTrail       | Enabled  | 363918845564         |
| IAM Identity Center  | Enabled  | 163209926128         |
| Region Deny Control  | Enabled  | eu-west-2 only       |
| AWS Backup           | Disabled | —                    |

---

## SCPs Applied

| SCP File                  | Applied To    | Purpose                              |
|---------------------------|---------------|--------------------------------------|
| region-lock.json          | Root          | Deny all activity outside eu-west-2  |
| deny-expensive-ec2.json   | Workloads OU  | Restrict to t2/t3/m5/c5/r5 families  |
| deny-public-s3.json       | Root          | Prevent public S3 buckets/ACLs       |
| deny-root-user.json       | Root          | Block all root user API activity     |

---

## Sign-in URLs

| Account           | URL                                                        |
|-------------------|------------------------------------------------------------|
| Management        | https://163209926128.signin.aws.amazon.com/console         |
| Config Aggregator | https://523835807624.signin.aws.amazon.com/console         |
| CloudTrail Admin  | https://363918845564.signin.aws.amazon.com/console         |
| SSO Portal        | Check IAM Identity Center after Control Tower completes    |

---

*Last updated: Day 02 — Control Tower deployment in progress*

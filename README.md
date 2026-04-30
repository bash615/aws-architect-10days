# Day 02 — AWS Organizations, Control Tower & Landing Zone

## What Was Built

AWS Control Tower Landing Zone deployed in eu-west-2 (London) with centralised
logging, Config aggregation, IAM Identity Center SSO, and region-lock guardrails.

## Account Structure

| Account           | ID           | Purpose               |
|-------------------|--------------|-----------------------|
| Bash (Management) | 163209926128 | Org management only   |
| Config Aggregator | 523835807624 | AWS Config aggregator |
| CloudTrail Admin  | 363918845564 | Centralised CT logs   |

## Files

```
day02/
  README.md                 — this file
  org-structure.md          — full OU tree and account inventory
  scps/
    region-lock.json        — deny all activity outside eu-west-2
    deny-expensive-ec2.json — restrict EC2 to approved instance families
    deny-public-s3.json     — prevent public S3 ACLs and bucket policies
    deny-root-user.json     — block all root user API activity in member accounts
```

## Services Enabled

- AWS Organizations (Management Account: 163209926128)
- AWS Control Tower (Home Region: eu-west-2)
- AWS Config (Aggregator: 523835807624)
- AWS CloudTrail centralised logging (Admin: 363918845564)
- IAM Identity Center / SSO
- Region deny control (eu-west-2 only)

## Key Decisions

- Region lock enabled at Control Tower setup — auto-applies SCP to all OUs
- Backup disabled — not needed at lab scale, can enable per-account later
- GuardDuty to be enabled post-setup via Security Hub
- Sandbox OU retained for future engineer sandbox accounts

## Next Steps (Day 03)

- VPC design: Hub-and-spoke with Transit Gateway
- Network segmentation: public/private/isolated subnets
- DNS: Route 53 Resolver with hybrid forwarding
- VPC Endpoints: private S3, ECR, SSM access

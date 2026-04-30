# Day 01 — AWS Foundation, IAM and Billing Safety

## Course Track

**AWS Platform Architect / AWS Cloud Architect 10-Day Course**

## Day 1 Goal

Build a safe AWS foundation before creating cloud resources.

The purpose of Day 1 is to understand how AWS accounts, identity, access and billing controls work. As an AWS Platform Architect, the first responsibility is not to deploy workloads immediately. The first responsibility is to create a secure and controlled foundation.

---

## What We Are Learning Today

Today we are covering:

- AWS account basics
- AWS Regions and Availability Zones
- Root account security
- Multi-factor authentication
- IAM users, groups, roles and policies
- IAM Identity Center
- Least privilege access
- AWS billing safety
- AWS Budgets
- Cost alerts
- Architect-level thinking for secure cloud foundations

---

## Why This Matters

In enterprise AWS architecture, everything starts with governance and identity.

Before deploying EC2, ECS, EKS, Lambda, S3 or databases, the platform needs:

- Secure access
- Controlled permissions
- Auditability
- Billing visibility
- Guardrails
- Separation of duties
- A clear operating model

A poorly secured AWS account can create security, compliance and cost risks very quickly.

---

## Architect Explanation

An AWS Platform Architect starts by designing a secure cloud foundation.

This means:

- Protecting the root account
- Enforcing MFA
- Creating controlled administrative access
- Avoiding long-term unnecessary credentials
- Using IAM roles where possible
- Applying least privilege
- Setting budgets and alerts
- Choosing a primary AWS region
- Documenting governance decisions

The goal is to create a safe starting point for future AWS workloads.

---

## Key AWS Concepts

### AWS Account

An AWS account is the security and billing boundary for AWS resources.

In enterprise environments, organisations normally use multiple AWS accounts, such as:

- Management account
- Security account
- Log archive account
- Shared services account
- Development account
- Test account
- Production account

For this course, we may use one AWS account, but we will design as if we understand multi-account enterprise patterns.

### Root User

The root user has full access to the AWS account.

Best practice:

- Do not use the root user for daily work
- Enable MFA on the root user
- Store root credentials securely
- Use IAM roles or IAM users for administration
- Monitor root account activity

### IAM

IAM stands for Identity and Access Management.

IAM controls who can access AWS and what actions they can perform.

IAM includes:

- Users
- Groups
- Roles
- Policies
- Permissions
- Trust relationships

### IAM User

An IAM user is an identity with credentials.

IAM users can have:

- Console password
- Access keys
- Attached policies
- Group memberships

In modern AWS architecture, IAM users should be used carefully. IAM roles and federated access are preferred in enterprise environments.

### IAM Role

An IAM role is an identity that can be assumed by a user, service or workload.

Roles are commonly used for:

- EC2 instance access
- Lambda execution
- Cross-account access
- CI/CD pipelines
- Kubernetes/EKS workload identity
- Temporary administrative access

Roles are preferred because they support temporary credentials.

### IAM Policy

An IAM policy defines permissions.

A policy says:

- What actions are allowed or denied
- Which resources those actions apply to
- Under what conditions access is allowed

Example permission areas:

- S3 read access
- EC2 start/stop access
- CloudWatch read access
- IAM admin access

### Least Privilege

Least privilege means giving only the permissions required to perform a task.

This is a core AWS security principle.

Architect-level examples:

- Developers should not have full admin access by default
- Applications should only access the S3 buckets they need
- CI/CD pipelines should only deploy to required environments
- Production access should be restricted and audited

### IAM Identity Center

IAM Identity Center provides centralised workforce access to AWS accounts and applications.

In enterprise AWS environments, it is commonly used to manage:

- User access
- Permission sets
- Multi-account access
- Single sign-on
- Role-based access to AWS accounts

This is important for AWS Platform Architect roles because it supports governed access at scale.

### AWS Budgets

AWS Budgets allows cost alerts to be created.

For this course, create a very small budget alert to avoid unexpected spend.

Recommended lab budget:

- £1 alert if you want strict control
- £5 alert if you want slightly more flexibility

---

## Day 1 Hands-On Tasks

### Task 1 — Sign in to AWS

Sign in to the AWS Console.

Use your normal AWS account.

Do not create paid resources today.

One-liner: This confirms you can access the AWS management plane.

### Task 2 — Confirm the AWS Region

Choose one primary region for this course.

Recommended region:

```text
eu-west-2
```

This is London.

One-liner: Using one primary region keeps the course simple and helps control cost.

### Task 3 — Enable MFA on the Root Account

Go to:

```text
AWS Console
Account name menu
Security credentials
Multi-factor authentication
Assign MFA device
```

Recommended option:

```text
Authenticator app
```

One-liner: MFA protects the most privileged identity in the AWS account.

### Task 4 — Review IAM

Go to:

```text
IAM
Users
Groups
Roles
Policies
```

Do not change anything yet unless we agree the exact structure.

One-liner: IAM is where AWS access and permissions are controlled.

### Task 5 — Review IAM Identity Center

Go to:

```text
IAM Identity Center
```

Check whether it is enabled.

Do not make major changes today.

One-liner: IAM Identity Center is used for centralised workforce access across AWS accounts.

### Task 6 — Create a Billing Budget Alert

Go to:

```text
Billing and Cost Management
Budgets
Create budget
Cost budget
```

Suggested values:

```text
Budget name: aws-platform-lab-budget
Budget amount: 1 GBP or 5 GBP
Alert threshold: 80 percent
Email: your email address
```

One-liner: AWS Budgets helps prevent unexpected cloud spend.

### Task 7 — Review Free Tier Usage

Go to:

```text
Billing and Cost Management
Free Tier
```

Check:

- Free Tier usage
- Current monthly spend
- Forecasted spend
- Any active services

One-liner: Free Tier monitoring helps keep the course low-cost.

---

## Day 1 Architecture Notes

For enterprise AWS architecture, Day 1 maps to the foundation layer.

The foundation layer includes:

- Account security
- Identity
- Access governance
- MFA
- Billing controls
- Budget alerts
- Region selection
- Documentation

This is the starting point for all later AWS platform design.

---

## AWS Platform Architect Talking Point

Use this in interviews:

> I start AWS platform design by securing the foundation first. That means protecting the root account with MFA, defining IAM and least-privilege access, reviewing IAM Identity Center for centralised access, selecting the operating region, and setting billing alerts before deploying workloads. This creates a controlled and auditable foundation for future cloud services.

---

## CV Sentence

> Designed secure AWS account foundation patterns covering MFA, IAM, IAM Identity Center, least privilege, billing controls, budgets and governance readiness.

---

## Day 1 Commands

Most of Day 1 is console-based.

No CLI commands are required yet.

Later in the course we will use:

```bash
aws sts get-caller-identity
aws configure
aws s3 ls
terraform plan
```

---

## Cost Control Reminder

Do not create expensive resources today.

Avoid:

- NAT Gateway
- EKS cluster
- Large EC2 instances
- RDS databases
- OpenSearch domains
- Always-on services

Safe today:

- MFA
- IAM review
- Budget alert
- Free Tier review
- Documentation

---

## Day 1 Completion Checklist

- [ ] Signed in to AWS Console
- [ ] Confirmed primary region as eu-west-2
- [ ] Enabled MFA on root account
- [ ] Reviewed IAM dashboard
- [ ] Reviewed IAM Identity Center
- [ ] Created AWS Budget alert
- [ ] Reviewed Free Tier usage
- [ ] Created Day 1 Markdown notes
- [ ] Understood why AWS foundation matters

---

## Summary

Today we focused on secure AWS foundations.

Before designing VPCs, EC2, ECS, EKS, Lambda or AI-enabled workloads, we need a safe account foundation.

Day 1 is about:

- Security first
- Identity first
- Cost control first
- Governance first

This is how an AWS Platform Architect thinks.

---

## Next Day

**Day 2 — AWS Organizations, Control Tower and Landing Zone Design**

We will cover:

- Multi-account architecture
- AWS Organizations
- Control Tower concepts
- SCP guardrails
- Security account
- Log archive account
- Dev/Test/Prod account separation
- Landing zone diagram

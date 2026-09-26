# AWS IAM — Identity & Access Management Lab

> **AWS Hands-On Project | IAM Users • Policies • Roles • MFA • Access Analyzer**

This folder documents a practical AWS IAM lab performed in the AWS Management Console.

The objective is to demonstrate how AWS identities are created, how permissions are defined with JSON policies, how policies are attached to identities, how IAM roles work, how MFA is configured, and how IAM Access Analyzer can be used to inspect external access.

---

## 1. Project Overview

AWS Identity and Access Management (IAM) controls **who can authenticate to AWS and what actions they are authorized to perform**.

This lab covers:

- AWS account and IAM dashboard review
- Root-account security checks
- IAM user creation
- IAM policy discovery
- Custom S3 read-only policy creation
- Policy attachment
- S3 permission concepts
- IAM roles
- Role permissions
- MFA configuration
- IAM Access Analyzer
- Least-privilege design
- GitHub-safe AWS documentation

### Lab flow

```text
AWS Account
    │
    ├── Root User
    │     ├── MFA
    │     └── No active root access keys
    │
    └── IAM
          │
          ├── User
          │     └── Permissions / Policies
          │
          ├── Policies
          │     └── S3 Read-Only JSON
          │
          ├── Roles
          │     └── Trust + Permissions
          │
          ├── MFA
          │
          └── Access Analyzer
```

---

# 2. Repository Structure

```text
01-IAM/
│
├── README.md
│
├── screenshots/
│   ├── 00-root-user-dashboard.png
│   ├── 01-iam-dashboard.png
│   ├── 02-test-user.png
│   ├── 03-policy-json.png
│   ├── 04-policy-permissions.png
│   ├── 05-policy-attached.png
│   ├── 06-allowed-action.png
│   ├── 07-access-denied.png
│   ├── 08-role.png
│   ├── 09-role-permissions.png
│   ├── 10-mfa.png
│   ├── 11-access-analyzer.png
│   └── 12-iam-user-dashboard.png
│
└── policy/
    └── s3-read-only.json
```

### Screenshot convention

The project keeps the requested filenames so that the GitHub structure remains consistent.

A few uploaded screenshots show **policy configuration screens rather than the final runtime test result**. Those cases are explicitly documented below instead of claiming that an action was successfully allowed or denied.

---

# 3. Evidence Index

| # | File | What the screenshot actually demonstrates |
|---|---|---|
| 00 | [`00-root-user-dashboard.png`](./screenshots/00-root-user-dashboard.png) | AWS Console Home / account environment |
| 01 | [`01-iam-dashboard.png`](./screenshots/01-iam-dashboard.png) | IAM Dashboard and security recommendations |
| 02 | [`02-test-user.png`](./screenshots/02-test-user.png) | IAM user creation configuration |
| 03 | [`03-policy-json.png`](./screenshots/03-policy-json.png) | Custom S3 read-only JSON policy |
| 04 | [`04-policy-permissions.png`](./screenshots/04-policy-permissions.png) | IAM Policies inventory |
| 05 | [`05-policy-attached.png`](./screenshots/05-policy-attached.png) | Policy attachment options during user creation |
| 06 | [`06-allowed-action.png`](./screenshots/06-allowed-action.png) | AmazonS3FullAccess managed policy inspection |
| 07 | [`07-access-denied.png`](./screenshots/07-access-denied.png) | Custom `s3:DeleteObject` policy configuration |
| 08 | [`08-role.png`](./screenshots/08-role.png) | IAM Roles inventory |
| 09 | [`09-role-permissions.png`](./screenshots/09-role-permissions.png) | Role permission attachment screen |
| 10 | [`10-mfa.png`](./screenshots/10-mfa.png) | IAM MFA device configuration |
| 11 | [`11-access-analyzer.png`](./screenshots/11-access-analyzer.png) | Access Analyzer external-access configuration |
| 12 | [`12-iam-user-dashboard.png`](./screenshots/12-iam-user-dashboard.png) | Separate AWS practice account / console environment |

---

# 4. Step 0 — AWS Console Environment

**Evidence:** [`00-root-user-dashboard.png`](./screenshots/00-root-user-dashboard.png)

The first screenshot shows the AWS Console Home.

Visible services include:

- IAM
- Billing and Cost Management
- AWS Health Dashboard
- EC2

The screenshot also shows the AWS Console region selector set to **Asia Pacific (Mumbai)**.

### Why this screenshot is useful

It establishes the AWS account environment before entering the IAM configuration workflow.

For a public GitHub repository, avoid documenting or exposing credentials even when the screenshot does not contain them.

---

# 5. Step 1 — IAM Dashboard

**Evidence:** [`01-iam-dashboard.png`](./screenshots/01-iam-dashboard.png)

The IAM Dashboard provides a central view of identity and access resources.

The supplied screenshot shows:

### Security recommendations

```text
Root user has MFA
Root user has no active access keys
```

This is important because AWS distinguishes the root user from IAM identities used for normal AWS operations.

### IAM resources shown

```text
User groups       1
Users             1
Roles             3
Policies          0
Identity providers 0
```

The screenshot also shows the account alias and IAM user sign-in URL.

### Main IAM areas visible

```text
Access Management
├── Roles
├── Policies
├── IAM users
├── IAM user groups
├── Identity providers
├── Account settings
└── Root access management

Access reports
├── Access Analyzer
├── Resource analysis
├── Unused access
├── Analyzer settings
├── Policy simulator
└── Credential report
```

---

# 6. Step 2 — Create an IAM User

**Evidence:** [`02-test-user.png`](./screenshots/02-test-user.png)

The screenshot shows the AWS IAM **Create user** workflow.

### User configured

```text
aws-doc-test-user
```

### Console access

The screenshot shows:

```text
Provide user access to the AWS Management Console
```

enabled.

The password configuration is set to:

```text
Autogenerated password
```

and:

```text
Users must create a new password at next sign-in
```

is enabled.

### Why create a dedicated IAM user?

A dedicated test identity allows permissions to be tested without performing every activity through the root user.

This makes the lab easier to reason about:

```text
Root / administrative identity
          │
          ▼
     IAM configuration
          │
          ▼
   Test IAM identity
          │
          ▼
     Permission tests
```

### Security rule

Never commit the generated password or other credentials to GitHub.

---

# 7. Step 3 — Custom S3 Read-Only Policy

**Evidence:** [`03-policy-json.png`](./screenshots/03-policy-json.png)

The screenshot shows the IAM JSON Policy Editor.

The custom policy is stored separately in:

[`policy/s3-read-only.json`](./policy/s3-read-only.json)

### Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowS3ReadOnly",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": "*"
    }
  ]
}
```

### Statement explanation

| Field | Value | Purpose |
|---|---|---|
| `Version` | `2012-10-17` | IAM policy language version |
| `Sid` | `AllowS3ReadOnly` | Statement identifier |
| `Effect` | `Allow` | Grants the specified actions |
| `Action` | `s3:GetObject` | Read an S3 object |
| `Action` | `s3:ListBucket` | List objects in a bucket |
| `Resource` | `*` | Applies broadly to matching resources |

### Important

This is a **learning policy**.

For production, replace:

```json
"Resource": "*"
```

with specific bucket/object ARNs wherever practical.

---

# 8. Step 4 — IAM Policies

**Evidence:** [`04-policy-permissions.png`](./screenshots/04-policy-permissions.png)

The screenshot shows the IAM **Policies** page.

AWS managed policies visible include examples such as:

- `AdministratorAccess`
- `AgentRegistryFullAccess`
- `AgentRegistryReadOnlyAccess`
- AI-related managed policies
- Account-management policies

### Policy categories

#### AWS managed policy

Maintained by AWS.

#### Customer managed policy

Created and maintained by the AWS account owner.

The S3 read-only policy in this project is designed as a custom policy for learning IAM JSON and permission scoping.

#### Inline policy

Embedded directly into a user, group, or role.

---

# 9. Step 5 — Attach Permissions to a User

**Evidence:** [`05-policy-attached.png`](./screenshots/05-policy-attached.png)

The screenshot shows the **Set permissions** step of IAM user creation.

AWS presents three approaches:

```text
1. Add user to group
2. Copy permissions
3. Attach policies directly
```

The screenshot has:

```text
Attach policies directly
```

selected.

### Recommended permission model

For repeatable permission management, groups can be used to represent a job function:

```text
IAM Group
    │
    ├── Policy A
    ├── Policy B
    └── Policy C
         │
         ▼
      IAM User
```

This reduces duplicated permission configuration when multiple users have the same responsibilities.

---

# 10. Step 6 — Inspect S3 Permissions

**Evidence:** [`06-allowed-action.png`](./screenshots/06-allowed-action.png)

### Important evidence clarification

The uploaded screenshot **does not show a successful S3 runtime action**.

It shows the AWS managed policy:

```text
AmazonS3FullAccess
```

and its policy document.

The screenshot states that the policy provides full access to S3 through the AWS Management Console.

The visible policy includes broad actions such as:

```text
s3:*
s3-object-lambda:*
```

with:

```text
Resource: *
```

### Why this matters

This screenshot provides a useful comparison with the custom project policy.

#### Broad access

```text
AmazonS3FullAccess
        │
        ├── s3:*
        └── s3-object-lambda:*
```

#### Project read-only access

```text
Custom Policy
        │
        ├── s3:GetObject
        └── s3:ListBucket
```

The second approach demonstrates a narrower permission set for the specific learning requirement.

> The filename `06-allowed-action.png` is retained for the requested project structure, but the screenshot itself should not be described as proof of a successful allowed-action test.

---

# 11. Step 7 — Custom DeleteObject Policy

**Evidence:** [`07-access-denied.png`](./screenshots/07-access-denied.png)

### Evidence clarification

The uploaded screenshot does **not** show an `AccessDenied` runtime error.

It shows an IAM JSON policy being created with:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowDeleteObjects",
      "Effect": "Allow",
      "Action": "s3:DeleteObject",
      "Resource": "arn:aws:s3:::ayush-aws-practice/*"
    }
  ]
}
```

### What this demonstrates

The policy grants:

```text
s3:DeleteObject
```

against objects under the specified bucket ARN.

This is useful for demonstrating how a single IAM action can be scoped to a specific resource.

### Important distinction

A policy definition is not the same as a runtime authorization result.

To document an actual denial test, the repository would need a screenshot showing an IAM principal receiving an `AccessDenied` response while attempting an action for which it has no effective permission.

---

# 12. Step 8 — IAM Roles

**Evidence:** [`08-role.png`](./screenshots/08-role.png)

The screenshot shows the IAM **Roles** page.

The account currently displays three service-linked roles:

```text
AWSServiceRoleForResourceExplorer
AWSServiceRoleForSupport
AWSServiceRoleForTrustedAdvisor
```

### What is an IAM role?

An IAM role is an identity that has permissions and can be assumed by a trusted entity.

Unlike an IAM user, a role is commonly used to provide **temporary credentials** to AWS services, workloads, or trusted principals.

### Common role use cases

```text
EC2
Lambda
ECS
EKS
Cross-account access
Federated users
AWS services
```

### Role model

```text
Trusted Entity
      │
      │ AssumeRole
      ▼
   IAM Role
      │
      ▼
Permissions Policies
      │
      ▼
AWS Resources
```

---

# 13. Step 9 — Role Permissions

**Evidence:** [`09-role-permissions.png`](./screenshots/09-role-permissions.png)

The screenshot shows the **Create role → Add permissions** step.

AWS provides:

```text
Use existing policy
```

or:

```text
Create inline policy
```

The screen lists managed policies that can be attached to the role.

### Trust policy vs permissions policy

These concepts should not be confused.

#### Trust policy

Answers:

> Who or what is allowed to assume this role?

#### Permissions policy

Answers:

> What can the role do after it is assumed?

```text
        Trust Policy
             │
             ▼
      Can this entity
       assume role?
             │
             ▼
          IAM Role
             │
             ▼
     Permissions Policy
             │
             ▼
       AWS resources
```

This distinction is fundamental to understanding IAM roles.

---

# 14. Step 10 — MFA

**Evidence:** [`10-mfa.png`](./screenshots/10-mfa.png)

The screenshot shows the IAM **Assign MFA device** workflow for the user:

```text
ayush-practice
```

The available MFA options shown include:

### Passkey or security key

Uses a passkey or FIDO2 security key.

### Authenticator app

Generates time-based verification codes through an authenticator application.

### Hardware TOTP token

Uses a dedicated hardware TOTP device.

The screenshot has:

```text
Authenticator app
```

selected.

### MFA model

```text
Username + Password
       +
      MFA
       │
       ▼
Authenticated session
```

### Security benefit

MFA provides an additional authentication factor, reducing reliance on the password alone.

---

# 15. Step 11 — IAM Access Analyzer

**Evidence:** [`11-access-analyzer.png`](./screenshots/11-access-analyzer.png)

The screenshot shows the **Create analyzer** page.

Selected finding type:

```text
Resource analysis — external access
```

Region:

```text
Asia Pacific (Mumbai)
```

### External-access analysis

This analyzer type examines supported resources and their policies to identify access from outside the configured zone of trust.

### Access Analyzer workflow

```text
AWS Resources
      │
      ▼
Access Analyzer
      │
      ▼
Policy analysis
      │
      ▼
Potential findings
      │
      ├── Review
      ├── Validate
      └── Remediate where appropriate
```

### Regional consideration

The supplied configuration is for:

```text
Asia Pacific (Mumbai)
```

If resources are used in other AWS regions, Access Analyzer configuration and findings should be reviewed for those regions as appropriate.

---

# 16. Step 12 — AWS Practice Account Console

**Evidence:** [`12-iam-user-dashboard.png`](./screenshots/12-iam-user-dashboard.png)

This screenshot shows a separate AWS practice environment using the account alias:

```text
ayush-practice
```

and region:

```text
Europe (Stockholm)
```

The AWS Console Home shows commonly used services:

```text
EC2
S3
CloudWatch
IAM
```

The AWS Health widget indicates that the current identity does not have permission to access AWS Health.

### Why this screenshot is useful

It demonstrates an important IAM concept:

> Access to the AWS Console does not automatically mean access to every AWS service.

A principal can have console access while still receiving authorization errors for services for which it has not been granted permission.

---

# 17. Permission Model

IAM authorization can be represented as:

```text
Request
   │
   ▼
Identity
   │
   ├── Identity policies
   ├── Resource policies
   ├── Permissions boundary
   ├── Session policies
   └── Organizational controls
          │
          ▼
     Authorization
          │
     ┌────┴────┐
     ▼         ▼
   Allow      Deny
```

### Core rule

An explicit `Deny` overrides an applicable `Allow`.

Also, an action generally requires an applicable permission grant; having access to the AWS Console alone does not grant permissions to every service.

---

# 18. Least Privilege

The project compares broad managed access with a narrower custom policy.

### Broad policy

```json
"Action": [
  "s3:*"
]
```

### Narrow project policy

```json
"Action": [
  "s3:GetObject",
  "s3:ListBucket"
]
```

The custom policy is intentionally limited to read/list operations.

### Resource scoping

The learning policy currently uses:

```json
"Resource": "*"
```

For a production policy, scope access to the exact:

```text
Bucket ARN
+
Object ARN/path
```

required by the workload.

---

# 19. Security Controls Demonstrated

| Control | Demonstration |
|---|---|
| Root MFA | IAM dashboard |
| No root access keys | IAM dashboard |
| Separate IAM user | `aws-doc-test-user` |
| Custom policy | S3 read-only JSON |
| Permission attachment | User permission screen |
| Policy inspection | AmazonS3FullAccess screenshot |
| Resource-specific action | `s3:DeleteObject` policy |
| IAM roles | Roles inventory |
| Role permissions | Add-permissions screen |
| MFA | Authenticator app configuration |
| Access analysis | External-access analyzer |
| Least privilege | Narrow S3 actions |

---

# 20. Custom Policy File

The repository contains:

[`policy/s3-read-only.json`](./policy/s3-read-only.json)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowS3ReadOnly",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": "*"
    }
  ]
}
```

### GitHub benefit

Keeping the policy in a separate JSON file makes the repository easier to review and allows the policy document to be inspected independently from the screenshots.

---

# 21. What This Project Demonstrates

### Identity

A dedicated IAM user can be created for AWS practice instead of relying on the root user for routine work.

### Authorization

IAM policies define the actions that an identity can perform.

### Resource-level permissions

Actions such as:

```text
s3:DeleteObject
```

can be scoped to a specific S3 resource ARN.

### Roles

IAM roles separate:

```text
Who can assume the identity?
```

from:

```text
What can the identity do?
```

### Authentication

MFA adds an additional authentication factor.

### Security analysis

IAM Access Analyzer can be used to inspect resource access relationships.

---

# 22. Project Outcomes

After completing this lab, the following practical IAM concepts are documented:

- IAM Dashboard navigation
- Root-account security checks
- IAM user creation
- Console access configuration
- Managed vs customer-created policies
- IAM JSON policy syntax
- S3 permissions
- Policy attachment
- IAM roles
- Trust policies vs permissions policies
- MFA
- IAM Access Analyzer
- Least privilege
- Resource-level authorization
- AWS Console permission boundaries

### Practical outcome

The project moves from:

```text
AWS Account
    ↓
IAM
    ↓
Identity
    ↓
Policy
    ↓
Resource
    ↓
Authorization
```

rather than treating AWS access as a single account-wide permission.

---

# 23. Important Evidence Note

Two filenames were originally intended to represent runtime tests:

```text
06-allowed-action.png
07-access-denied.png
```

The supplied screenshots currently show:

```text
06 → AmazonS3FullAccess policy details
07 → s3:DeleteObject custom policy creation
```

They do **not** show the final runtime results of an allowed request and an `AccessDenied` request.

This README intentionally describes the screenshots accurately rather than presenting configuration evidence as runtime evidence.

### If you later perform the actual tests

Capture:

```text
06-allowed-action.png
→ successful S3 ListBucket/GetObject operation

07-access-denied.png
→ actual AccessDenied result for an unpermitted action
```

The filenames can remain unchanged.

---

# 24. GitHub Security Checklist

Before pushing this project publicly:

```text
[ ] No AWS passwords
[ ] No access keys
[ ] No secret access keys
[ ] No session tokens
[ ] No private keys
[ ] No MFA recovery codes
[ ] No .env files containing secrets
[ ] No ~/.aws/credentials file
[ ] No credentials copied into README
[ ] Review screenshots for accidental secrets
```

### Recommended `.gitignore`

```gitignore
.env
.env.*
.aws/
*.pem
*.key
credentials
*.secret
secrets/
```

### Important

AWS account IDs, aliases, and sign-in URLs are not equivalent to passwords or secret access keys, but review public screenshots before publishing and remove anything you do not want exposed.

---

# 25. Cleanup

After the lab:

```text
[ ] Remove unused test IAM users
[ ] Remove unnecessary policy attachments
[ ] Delete unused customer-managed policies
[ ] Remove unused roles
[ ] Remove unnecessary Access Analyzer analyzers
[ ] Check for unused access keys
[ ] Verify billing/cost dashboard
```

Do not delete an IAM identity without checking whether another AWS resource or workflow depends on it.

---

# 26. Git Commands

From the repository root:

```bash
git add 02-iam/
git commit -m "docs: add AWS IAM hands-on lab"
git push origin main
```

Check the repository before pushing:

```bash
git status
git diff --cached
```

---

# 27. Final Project Tree

```text
02-iam/
│
├── README.md
│
├── screenshots/
│   ├── 00-root-user-dashboard.png
│   ├── 01-iam-dashboard.png
│   ├── 02-test-user.png
│   ├── 03-policy-json.png
│   ├── 04-policy-permissions.png
│   ├── 05-policy-attached.png
│   ├── 06-allowed-action.png
│   ├── 07-access-denied.png
│   ├── 08-role.png
│   ├── 09-role-permissions.png
│   ├── 10-mfa.png
│   ├── 11-access-analyzer.png
│   └── 12-iam-user-dashboard.png
│
└── policy/
    └── s3-read-only.json
```

---

# 28. Portfolio Summary

> **AWS IAM Hands-On Security Lab**
>
> Built and documented an AWS IAM security lab covering IAM users, custom JSON policies, S3 permissions, policy attachment, IAM roles, MFA, and IAM Access Analyzer.
>
> The project demonstrates practical cloud-security concepts including identity management, authorization, least privilege, resource-level permissions, role-based access, multi-factor authentication, and access analysis.
>
> Documentation includes AWS Console evidence, reusable IAM policy JSON, architecture, testing concepts, security controls, outcomes, and GitHub-safe credential practices.

---

## Conclusion

This IAM project establishes the identity and authorization foundation for the rest of the AWS hands-on repository.

```text
                    AWS
                     │
                     ▼
                    IAM
                     │
        ┌────────────┼────────────┐
        ▼            ▼            ▼
      Users        Roles       Policies
        │            │            │
        └────────────┼────────────┘
                     ▼
                  Access
                     │
          ┌──────────┴──────────┐
          ▼                     ▼
        Allow                  Deny
          │                     │
          ▼                     ▼
       Resource              Blocked
          │
          ▼
   Security Monitoring
          │
          ▼
   Access Analyzer
```

**Project status:** IAM documentation and supplied screenshot evidence are organized and linked. Runtime allow/deny screenshots can be replaced later if you perform those exact tests.

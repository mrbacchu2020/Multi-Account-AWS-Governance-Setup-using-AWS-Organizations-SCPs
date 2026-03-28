# Multi-Account-AWS-Governance-Setup-using-AWS-Organizations-SCPs
Built a centralized AWS governance model using AWS Organizations, OUs, and SCPs to enforce security, cost, and compliance across Dev, Test, and Prod accounts by restricting EC2 usage, protecting CloudTrail, and limiting regions
---
## 📌 Project Objective

This project simulates an enterprise cloud environment with separate AWS accounts for:

* Development Team
* Testing Team
* Production Team

The goal is to implement centralized governance so that:

* Developers cannot launch expensive EC2 instances in Dev
* CloudTrail cannot be disabled or deleted
* AWS usage is restricted to approved regions only

---

# 🏗️ Architecture Diagram

📁 Path: `docs/architecture-diagram.png`

![Architecture](docs/architecture-diagram.png)

---

# ⚙️ Technologies Used

* AWS Organizations
* Organizational Units (OUs)
* Service Control Policies (SCPs)
* AWS IAM
* AWS CloudTrail
* Amazon EC2

---

# 📂 Project Structure

```text
multi-account-aws-governance/
│
├── README.md
│
├── docs/
│   └── architecture-diagram.png
│
├── screenshots/
│   ├── 01-organization-created.png
│   ├── 02-ou-structure-created.png
│   ├── 03-member-accounts-added.png
│   ├── 04-final-ou-account-structure.png
│   ├── 05-scp-enabled.png
│   ├── 06-dev-ec2-scp-json.png
│   ├── 07-dev-ec2-scp-attached.png
│   ├── 08-cloudtrail-protection-scp-json.png
│   ├── 09-cloudtrail-protection-attached-root.png
│   ├── 10-region-restriction-scp-json.png
│   ├── 11-region-restriction-attached-root.png
│   ├── 12-cloudtrail-org-trail-config.png
│   ├── 13-cloudtrail-org-trail-created.png
│   ├── 14-dev-ec2-restricted-instance-attempt.png
│   ├── 15-dev-ec2-access-denied-proof.png
│   ├── 16-dev-allowed-instance-success.png
│   ├── 17-cloudtrail-visible-member-account.png
│   ├── 18-cloudtrail-stop-delete-attempt.png
│   ├── 19-cloudtrail-access-denied-proof.png
│   ├── 20-unapproved-region-selected.png
│   ├── 21-region-restriction-access-denied.png
│   └── 22-cloudtrail-denied-event-history.png
│
└── scp-policies/
    ├── deny-large-ec2-dev.json
    ├── prevent-cloudtrail-disable.json
    └── restrict-regions.json
```

---

# 🚀 Step-by-Step Implementation

---

## Step 1 — Create AWS Organization

### Action

Login to your **AWS Management Account (Root Account)** and open:

```text
AWS Console → AWS Organizations
```

### Perform

* Click **Create organization**
* Choose **All Features**
* Finish setup

### Why

This enables advanced governance features such as **Service Control Policies (SCPs)**.

📸 **Screenshot:**
`screenshots/01-organization-created.png`

---

## Step 2 — Create Organizational Units (OUs)

### Action

Inside AWS Organizations, create the following OUs:

* Dev-OU
* Test-OU
* Prod-OU

### Perform

* Open **Root**
* Click **Actions**
* Select **Create new organizational unit**
* Repeat for all 3 OUs

📸 **Screenshot:**
`screenshots/02-ou-structure-created.png`

---

## Step 3 — Add Member Accounts

### Action

Create or invite AWS accounts for:

* Dev Account
* Test Account
* Prod Account

### Perform

Go to:

```text
AWS Organizations → Accounts → Add an AWS account
```

Choose either:

* **Create AWS account**
* **Invite existing AWS account**

📸 **Screenshot:**
`screenshots/03-member-accounts-added.png`

---

## Step 4 — Move Accounts into Correct OUs

### Action

Move each account into its appropriate OU.

### Mapping

* Dev Account → Dev-OU
* Test Account → Test-OU
* Prod Account → Prod-OU

### Perform

* Select account
* Click **Actions**
* Choose **Move**

📸 **Screenshot:**
`screenshots/04-final-ou-account-structure.png`

---

## Step 5 — Enable Service Control Policies (SCPs)

### Action

Open:

```text
AWS Organizations → Policies
```

### Perform

* Check **Service Control Policies**
* If disabled, click **Enable**

📸 **Screenshot:**
`screenshots/05-scp-enabled.png`

---

# 🔐 SCP Policies

---

## Step 6 — Create SCP: Deny Large EC2 in Dev

### Purpose

Restrict developers from launching expensive EC2 instances in Dev account.

### File

📁 `scp-policies/deny-large-ec2-dev.json`

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyLargeEC2InstancesInDev",
      "Effect": "Deny",
      "Action": "ec2:RunInstances",
      "Resource": "*",
      "Condition": {
        "StringNotEquals": {
          "ec2:InstanceType": [
            "t2.micro",
            "t2.small",
            "t3.micro",
            "t3.small"
          ]
        }
      }
    }
  ]
}
```

### Attach To

* **Dev-OU**

📸 **Screenshots:**

* `screenshots/06-dev-ec2-scp-json.png`
* `screenshots/07-dev-ec2-scp-attached.png`

---

## Step 7 — Create SCP: Prevent CloudTrail Disable/Delete

### Purpose

Prevent users from disabling or deleting CloudTrail logs.

### File

📁 `scp-policies/prevent-cloudtrail-disable.json`

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyCloudTrailDisableDelete",
      "Effect": "Deny",
      "Action": [
        "cloudtrail:StopLogging",
        "cloudtrail:DeleteTrail",
        "cloudtrail:UpdateTrail",
        "cloudtrail:PutEventSelectors",
        "cloudtrail:PutInsightSelectors"
      ],
      "Resource": "*"
    }
  ]
}
```

### Attach To

* **Root**

📸 **Screenshots:**

* `screenshots/08-cloudtrail-protection-scp-json.png`
* `screenshots/09-cloudtrail-protection-attached-root.png`

---

## Step 8 — Create SCP: Restrict AWS Regions

### Purpose

Restrict all accounts to approved AWS regions only.

### Approved Regions

* `ap-south-1`
* `us-east-1`

### File

📁 `scp-policies/restrict-regions.json`

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyUnapprovedRegions",
      "Effect": "Deny",
      "NotAction": [
        "iam:*",
        "organizations:*",
        "route53:*",
        "cloudfront:*",
        "support:*",
        "budgets:*",
        "ce:*",
        "sts:*"
      ],
      "Resource": "*",
      "Condition": {
        "StringNotEquals": {
          "aws:RequestedRegion": [
            "ap-south-1",
            "us-east-1"
          ]
        }
      }
    }
  ]
}
```

### Attach To

* **Root**

📸 **Screenshots:**

* `screenshots/10-region-restriction-scp-json.png`
* `screenshots/11-region-restriction-attached-root.png`

---

# 📋 CloudTrail Setup

---

## Step 9 — Create Organization CloudTrail

### Action

Go to:

```text
AWS Console → CloudTrail
```

### Perform

* Click **Create trail**
* Trail name:

```text
org-governance-trail
```

* Choose:

  * **Apply trail to my organization**
  * **Multi-region trail**
  * **Enable log validation**
* Store logs in S3 bucket

📸 **Screenshots:**

* `screenshots/12-cloudtrail-org-trail-config.png`
* `screenshots/13-cloudtrail-org-trail-created.png`

---

# 🧪 Validation & Testing

---

## Step 10 — Validate Dev EC2 Restriction

### Action

Login to **Dev Account**

Go to:

```text
EC2 → Launch Instance
```

### Perform

Try launching:

```text
t3.large
```

or

```text
m5.large
```

### Expected Result

❌ Access Denied

📸 **Screenshots:**

* `screenshots/14-dev-ec2-restricted-instance-attempt.png`
* `screenshots/15-dev-ec2-access-denied-proof.png`

---

## Step 11 — Validate Allowed EC2 in Dev

### Action

Still in Dev account, launch:

```text
t2.micro
```

or

```text
t3.micro
```

### Expected Result

✅ Instance launches successfully

📸 **Screenshot:**
`screenshots/16-dev-allowed-instance-success.png`

---

## Step 12 — Validate CloudTrail Protection

### Action

From Dev/Test/Prod account, go to:

```text
CloudTrail → Trails
```

### Perform

Try:

* Stop logging
* Delete trail
* Modify trail

### Expected Result

❌ Access Denied

📸 **Screenshots:**

* `screenshots/17-cloudtrail-visible-member-account.png`
* `screenshots/18-cloudtrail-stop-delete-attempt.png`
* `screenshots/19-cloudtrail-access-denied-proof.png`

---

## Step 13 — Validate Region Restriction

### Action

Switch AWS region to an unapproved region such as:

```text
eu-west-1
```

### Perform

Try creating:

* EC2 instance
* VPC
* S3 bucket

### Expected Result

❌ Access Denied

📸 **Screenshots:**

* `screenshots/20-unapproved-region-selected.png`
* `screenshots/21-region-restriction-access-denied.png`

---

## Step 14 — Validate in CloudTrail Event History

### Action

Open:

```text
CloudTrail → Event history
```

### Search For

* RunInstances
* StopLogging
* DeleteTrail

### Expected Result

Events appear with denied access logs

📸 **Screenshot:**
`screenshots/22-cloudtrail-denied-event-history.png`

---

# ✅ Validation Summary

| Test Case               | Expected Result | Status |
| ----------------------- | --------------- | ------ |
| Launch large EC2 in Dev | Access Denied   | ✅      |
| Launch small EC2 in Dev | Allowed         | ✅      |
| Stop/Delete CloudTrail  | Access Denied   | ✅      |
| Use unapproved region   | Access Denied   | ✅      |

---

# 🔐 Governance Benefits

## 💰 Cost Control

Prevents unnecessary use of expensive resources in Dev.

## 🔒 Security

Ensures CloudTrail remains enabled for audit and investigation.

## 🌍 Compliance

Restricts infrastructure usage to approved AWS regions only.

## 🧭 Centralized Governance

Allows all controls to be managed from a single management account.

---

# ⭐ Key Learnings

* SCPs do **not grant permissions**
* SCPs define **maximum allowed permissions**
* Explicit deny in SCP overrides IAM permissions
* AWS Organizations simplifies multi-account governance

---

# 📌 Conclusion

This project demonstrates how to implement enterprise-level AWS governance using AWS Organizations and Service Control Policies. It helps improve **security**, **cost optimization**, and **compliance** across multiple AWS accounts in a centralized and scalable manner.

---

# 👨‍💻 Author

**Shreyash Meshram**
AWS Cloud & DevOps Engineer

---

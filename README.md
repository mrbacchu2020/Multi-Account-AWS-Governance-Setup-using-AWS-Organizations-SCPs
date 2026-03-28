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


<img width="1536" height="1024" alt="ChatGPT Image Mar 28, 2026, 10_47_11 AM" src="https://github.com/user-attachments/assets/7d853c91-b69e-4abb-8847-ecd1acb152e5" />

---

# ⚙️ Technologies Used

* AWS Organizations
* Organizational Units (OUs)
* Service Control Policies (SCPs)
* AWS IAM
* AWS CloudTrail
* Amazon EC2

---

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

<img width="1920" height="1200" alt="Screenshot 2026-03-11 160203" src="https://github.com/user-attachments/assets/40f2d9f4-e8ae-4e3d-a073-dbae384a7f1c" />
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

<img width="1862" height="1126" alt="Screenshot 2026-03-28 112101" src="https://github.com/user-attachments/assets/997b9fa1-1070-43c4-a252-413ad33d597b" />

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

<img width="1869" height="1143" alt="Screenshot 2026-03-28 112624" src="https://github.com/user-attachments/assets/3742fcc2-e6cc-497c-a1a6-dea8afeddd79" />
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

<img width="1920" height="1200" alt="Screenshot 2026-03-11 163012" src="https://github.com/user-attachments/assets/d66d293f-11da-48ae-bca4-005bd06fb921" />
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
<img width="1876" height="1140" alt="Screenshot 2026-03-28 114653" src="https://github.com/user-attachments/assets/d2a75ffc-d4f2-4141-8252-51651e05c455" />
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

<img width="1891" height="1145" alt="Screenshot 2026-03-28 114807" src="https://github.com/user-attachments/assets/04f529a5-f10e-454c-bc48-e420beb27fa9" />
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

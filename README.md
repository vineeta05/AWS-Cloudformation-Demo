# 🚀 AWS EC2 SSM Access via CloudFormation & GitHub Actions

![AWS](https://img.shields.io/badge/AWS-CloudFormation-orange?logo=amazon-aws)
![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-CI%2FCD-blue?logo=github-actions)
![SSM](https://img.shields.io/badge/AWS-SSM_Session_Manager-yellow?logo=amazon-aws)
![IaC](https://img.shields.io/badge/Infrastructure-as_Code-green)

A complete **Infrastructure as Code (IaC)** project that provisions an EC2 instance on AWS and connects to it securely using **AWS Systems Manager (SSM) Session Manager** — no SSH keys, no open ports required.

---

## 📌 Project Overview

This project demonstrates two ways to deploy AWS infrastructure using CloudFormation:

| Method | Description |
|---|---|
| **Way 1** | Upload CloudFormation template manually from local machine |
| **Way 2** | Automated CI/CD pipeline via GitHub Actions → S3 → CloudFormation |

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────┐
│                     GitHub Repository                    │
│                                                          │
│   ec2-ssm.yaml          .github/workflows/deploy.yml    │
└────────────┬────────────────────────┬────────────────────┘
             │                        │
             │ Way 1 (Manual)         │ Way 2 (Automated)
             │                        ▼
             │              ┌─────────────────┐
             │              │  GitHub Actions  │
             │              │   (OIDC Auth)    │
             │              └────────┬────────┘
             │                       │
             │                       ▼
             │              ┌─────────────────┐
             │              │    AWS S3 Bucket │
             │              │  (Template Store)│
             │              └────────┬────────┘
             │                       │
             ▼                       ▼
      ┌──────────────────────────────────────┐
      │         AWS CloudFormation           │
      │  (Reads template, creates resources) │
      └──────────────────┬───────────────────┘
                         │ Creates
                         ▼
      ┌──────────────────────────────────────┐
      │            EC2 Instance              │
      │   ┌──────────────────────────────┐   │
      │   │     IAM Role (SSM Access)    │   │
      │   └──────────────────────────────┘   │
      │   ┌──────────────────────────────┐   │
      │   │    Security Group (443 out)  │   │
      │   └──────────────────────────────┘   │
      │   ┌──────────────────────────────┐   │
      │   │       SSM Agent              │   │
      │   └──────────────────────────────┘   │
      └──────────────────┬───────────────────┘
                         │
                         ▼
      ┌──────────────────────────────────────┐
      │       AWS SSM Session Manager        │
      │    (Secure shell, no SSH needed!)    │
      └──────────────────────────────────────┘
```

---

## 🛠️ AWS Resources Created

| Resource | Purpose |
|---|---|
| **EC2 Instance** | Virtual server (t3.micro, Amazon Linux 2) |
| **IAM Role** | Grants EC2 permission to use SSM |
| **IAM Instance Profile** | Attaches IAM role to EC2 |
| **Security Group** | Allows only outbound HTTPS (443) — no inbound SSH |
| **SSM Session Manager** | Secure browser/CLI-based terminal access |

---

## 🔐 Security Highlights

- ✅ **No SSH keys** required
- ✅ **No open inbound ports** (port 22 closed)
- ✅ **No hardcoded AWS credentials** — uses OIDC for GitHub Actions
- ✅ **Temporary tokens only** — GitHub gets 1-hour access tokens via OIDC
- ✅ **All secrets stored** in GitHub Secrets, never in code

---

## 📋 Prerequisites

- AWS Account
- GitHub Account
- AWS CLI installed locally
- Session Manager Plugin installed (for CLI access)

---

## 🚀 Way 1 — Deploy from Local Machine

### Step 1 — Clone the repo
```bash
git clone https://github.com/vineeta05/AWS-Cloudformation-Demo.git
cd AWS-Cloudformation-Demo
```

### Step 2 — Deploy via AWS Console
1. Go to **AWS Console → CloudFormation → Create Stack**
2. Upload `ec2-ssm.yaml`
3. Fill in parameters:
   - `VpcId` — Your VPC ID
   - `SubnetId` — A public subnet ID
   - `AssignPublicIp` — `true`
4. Click through and **Submit**

### Step 3 — Connect via SSM
```bash
aws ssm start-session --target <instance-id>
```

---

## ⚙️ Way 2 — Automated GitHub Actions Pipeline

### Step 1 — Set up IAM OIDC Identity Provider
- Provider URL: `https://token.actions.githubusercontent.com`
- Audience: `sts.amazonaws.com`

### Step 2 — Create GitHubActionsRole with these policies:
- `AmazonS3FullAccess`
- `AWSCloudFormationFullAccess`
- `IAMFullAccess`
- `AmazonEC2FullAccess`
- `AmazonSSMReadOnlyAccess`

### Step 3 — Add GitHub Secrets

| Secret | Value |
|---|---|
| `AWS_ROLE_ARN` | ARN of GitHubActionsRole |
| `AWS_REGION` | `us-east-1` |
| `VPC_ID` | Your VPC ID |
| `SUBNET_ID` | Your public Subnet ID |

### Step 4 — Push to main branch
The workflow triggers automatically and:
1. Authenticates to AWS via OIDC
2. Uploads template to S3
3. Deploys CloudFormation stack
4. Prints EC2 Instance ID and SSM connect command

---

## 📁 Project Structure

```
AWS-Cloudformation-Demo/
│
├── ec2-ssm.yaml                    # CloudFormation template
├── .github/
│   └── workflows/
│       └── deploy.yml              # GitHub Actions workflow
└── README.md                       # This file
```

---

## 💡 Key Concepts Learned

| Concept | Description |
|---|---|
| **Infrastructure as Code** | Define AWS resources in YAML instead of clicking |
| **IAM Roles & Policies** | Control what AWS services can do |
| **OIDC Authentication** | Passwordless auth between GitHub and AWS |
| **SSM Session Manager** | Secure EC2 access without SSH |
| **CI/CD Pipeline** | Automated deployment on every code push |

---

## 🧹 Cleanup

To avoid AWS charges, delete resources in this order:
```bash
# Delete CloudFormation stack (removes EC2, IAM Role, Security Group)
aws cloudformation delete-stack --stack-name vineeta-ssm-ec2-stack

# Empty and delete S3 bucket (optional)
aws s3 rm s3://vineeta-devops-bucket --recursive
```



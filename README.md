# 🔒 Security-Compliance-Guardrail-Lab

![AWS](https://img.shields.io/badge/AWS-%23FF9900.svg?style=for-the-badge&logo=amazon-aws&logoColor=white)
![Python](https://img.shields.io/badge/python-3670A0?style=for-the-badge&logo=python&logoColor=ffdd54)
![Terraform](https://img.shields.io/badge/terraform-%235835CC.svg?style=for-the-badge&logo=terraform&logoColor=white)
![Security](https://img.shields.io/badge/Security-Critical-red?style=for-the-badge)
![License](https://img.shields.io/badge/license-MIT-green?style=for-the-badge)

> **Hands-on AWS security lab demonstrating real-world misconfiguration detection, automated remediation, and compliance enforcement—built to showcase cloud security skills for entry-level DevOps and Cloud Engineering roles.**

---

## 🎯 Project Overview

This lab simulates a production AWS environment where security misconfigurations are intentionally introduced and then detected and remediated using automated guardrails. It demonstrates practical knowledge of:

- **AWS Security Best Practices** - IAM policies, S3 bucket security, encryption enforcement
- **Automated Compliance** - Event-driven remediation using Lambda and CloudWatch
- **Incident Response** - CloudTrail logging, alerting, and forensic analysis
- **Infrastructure as Code** - Terraform deployment and configuration management
- **Preventative CloudOps** - Proactive security controls and monitoring

**Perfect for**: Cloud Security roles, DevOps positions, Cloud Operations, and entry-level AWS jobs requiring security awareness.

---

## 🏗️ Architecture

```
┌─────────────────┐
│   CloudTrail    │──► Logs all API calls
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  EventBridge    │──► Detects misconfigurations
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Lambda Function│──► Auto-remediates issues
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  SNS Alerts     │──► Notifies security team
└─────────────────┘
```

### Monitored Resources:
- **S3 Buckets** - Public access, encryption status, versioning
- **IAM Users** - Overly permissive policies, unused credentials
- **EC2 Instances** - Security group misconfigurations, missing tags
- **RDS Databases** - Public accessibility, backup configurations

---

## 🚀 Features

### ✅ Automated Guardrails
- **Public S3 Detection** - Instantly detects and locks down publicly accessible buckets
- **Encryption Enforcement** - Enables default encryption on non-compliant S3 buckets
- **IAM Policy Validation** - Flags overly permissive wildcard permissions (`*`)
- **Security Group Hardening** - Alerts on 0.0.0.0/0 inbound rules on sensitive ports

### 📊 Monitoring & Alerting
- Real-time CloudWatch dashboards for security metrics
- SNS email/SMS notifications for critical violations
- CloudTrail integration for audit trails and forensics
- Custom CloudWatch Logs for compliance reporting

### 🔧 Remediation Workflows
1. **Detect** - EventBridge rule triggers on CloudTrail events
2. **Analyze** - Lambda function evaluates resource configuration
3. **Remediate** - Automated fix applied (e.g., block public access)
4. **Alert** - Security team notified via SNS
5. **Log** - Action recorded in CloudWatch for compliance

---

## 🛠️ Tech Stack

| Category | Tools |
|----------|-------|
| **Cloud Provider** | AWS (IAM, S3, Lambda, CloudTrail, EventBridge, SNS, CloudWatch) |
| **IaC** | Terraform |
| **Scripting** | Python 3.9+ (Boto3 SDK) |
| **CI/CD** | GitHub Actions |
| **Monitoring** | CloudWatch, CloudTrail |

---

## 📋 Prerequisites

- AWS account with appropriate IAM permissions
- AWS CLI configured (`aws configure`)
- Terraform installed (v1.0+)
- Python 3.9+ with Boto3 library
- Basic understanding of AWS security services

---

## ⚡ Quick Start

### 1. Clone the Repository
```bash
git clone https://github.com/charles-bucher/Security-Compliance-Guardrail-Lab.git
cd Security-Compliance-Guardrail-Lab
```

### 2. Set Up AWS Credentials
```bash
export AWS_ACCESS_KEY_ID="your-access-key"
export AWS_SECRET_ACCESS_KEY="your-secret-key"
export AWS_DEFAULT_REGION="us-east-1"
```

### 3. Deploy Infrastructure with Terraform
```bash
cd terraform
terraform init
terraform plan
terraform apply -auto-approve
```

### 4. Test Misconfiguration Detection
```bash
# Create a public S3 bucket (intentional misconfiguration)
aws s3api create-bucket --bucket test-public-bucket-$(date +%s) --acl public-read

# Watch CloudWatch Logs for automated remediation
aws logs tail /aws/lambda/guardrail-remediation --follow
```

### 5. Verify Remediation
Check your email (SNS) and CloudWatch dashboard to confirm:
- Lambda detected the public bucket
- Public access was automatically blocked
- Alert was sent to security team

---

## 📁 Project Structure

```
Security-Compliance-Guardrail-Lab/
│
├── terraform/                  # Infrastructure as Code
│   ├── main.tf                # Core AWS resources
│   ├── variables.tf           # Configuration variables
│   ├── outputs.tf             # Stack outputs
│   └── modules/
│       ├── guardrails/        # Lambda functions and EventBridge rules
│       ├── monitoring/        # CloudWatch dashboards and alarms
│       └── iam/               # IAM roles and policies
│
├── lambda/                    # Python remediation functions
│   ├── s3_guardrails.py       # S3 security automation
│   ├── iam_guardrails.py      # IAM policy validation
│   └── ec2_guardrails.py      # EC2 security group checks
│
├── tests/                     # Unit and integration tests
│   ├── test_s3_remediation.py
│   └── test_event_triggers.py
│
├── docs/                      # Additional documentation
│   ├── ARCHITECTURE.md        # Detailed architecture diagrams
│   ├── RUNBOOK.md            # Incident response procedures
│   └── COMPLIANCE.md         # Compliance framework mapping
│
└── README.md                  # You are here!
```

---

## 🔍 Use Cases Demonstrated

### 1. **S3 Public Access Remediation**
**Scenario**: Developer accidentally makes S3 bucket public  
**Detection**: CloudTrail logs `PutBucketAcl` API call  
**Remediation**: Lambda automatically applies `BlockPublicAccess`  
**Outcome**: Data breach prevented, team notified

### 2. **Unencrypted S3 Bucket**
**Scenario**: Bucket created without default encryption  
**Detection**: EventBridge rule triggers on `CreateBucket`  
**Remediation**: Lambda enables AES-256 encryption  
**Outcome**: Compliance requirement met automatically

### 3. **Overly Permissive IAM Policy**
**Scenario**: IAM role created with `s3:*` on `Resource: *`  
**Detection**: Custom Lambda evaluates IAM policy changes  
**Remediation**: Alert sent to security team for manual review  
**Outcome**: Least privilege principle enforced

---

## 📊 Compliance Frameworks Aligned

This lab demonstrates controls relevant to:

- **NIST Cybersecurity Framework** - Identify, Protect, Detect, Respond, Recover
- **CIS AWS Foundations Benchmark** - Section 2 (Storage), Section 1 (IAM)
- **AWS Well-Architected Framework** - Security Pillar
- **SOC 2 Type II** - Monitoring and incident response capabilities

---

## 🎓 Skills Demonstrated

✅ **Cloud Security** - Implementing preventative and detective controls  
✅ **Automation** - Event-driven workflows with Lambda and EventBridge  
✅ **Infrastructure as Code** - Terraform for repeatable deployments  
✅ **Scripting** - Python with Boto3 for AWS automation  
✅ **Monitoring** - CloudWatch dashboards, logs, and alarms  
✅ **Incident Response** - Automated remediation and alerting  
✅ **Compliance** - Mapping technical controls to frameworks  

---

## 🧪 Testing

Run automated tests to verify guardrails:

```bash
# Install dependencies
pip install -r requirements.txt

# Run unit tests
python -m pytest tests/ -v

# Test S3 remediation workflow
python tests/test_s3_remediation.py
```

**Expected Output**:
```
✓ S3 public access detected
✓ Lambda remediation triggered
✓ Public access blocked
✓ SNS alert sent
```

---

## 📈 Roadmap

### Phase 1 (Current)
- [x] S3 security guardrails
- [x] IAM policy validation
- [x] CloudTrail logging
- [x] Basic Lambda remediation

### Phase 2 (In Progress)
- [ ] EC2 security group automation
- [ ] RDS backup compliance checks
- [ ] Multi-region support
- [ ] Cost optimization alerts

### Phase 3 (Planned)
- [ ] Integration with AWS Security Hub
- [ ] Custom Config Rules
- [ ] Automated threat response with GuardDuty
- [ ] Compliance reporting dashboard

---

## 🤝 Contributing

Contributions are welcome! Feel free to:
- Report bugs via Issues
- Suggest new guardrails or detection rules
- Submit pull requests with improvements

---

## 📞 Connect With Me

Building hands-on AWS cloud projects to break into DevOps and Cloud Security roles.

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white)](https://linkedin.com/in/charles-bucher-cloud)
[![GitHub](https://img.shields.io/badge/GitHub-100000?style=for-the-badge&logo=github&logoColor=white)](https://github.com/charles-bucher)

💼 **Open to opportunities**: Entry-level Cloud Engineer, DevOps, AWS Security roles  
🎯 **Certifications**: AWS Solutions Architect Associate (in progress)  
📍 **Location**: Largo, Florida | Open to remote

---

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## ⭐ Show Your Support

If this project helped you learn AWS security or build your own cloud portfolio, give it a ⭐️!

---

**Built with ☕ and ☁️ by Charles Bucher**  
*Demonstrating real-world cloud security skills through hands-on projects*
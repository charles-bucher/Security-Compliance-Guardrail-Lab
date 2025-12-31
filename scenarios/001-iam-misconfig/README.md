# 🔐 Scenario 001: IAM Misconfiguration Detection & Remediation

![Difficulty](https://img.shields.io/badge/Difficulty-Intermediate-yellow?style=for-the-badge)
![AWS](https://img.shields.io/badge/AWS-IAM-FF9900?style=for-the-badge&logo=amazon-aws)
![Security](https://img.shields.io/badge/Security-Critical-red?style=for-the-badge)
![Duration](https://img.shields.io/badge/Duration-30--45_min-blue?style=for-the-badge)

## 📋 Overview

This scenario simulates real-world IAM security misconfigurations commonly found in production AWS environments. You'll practice detecting overly permissive policies, unused credentials, missing MFA enforcement, and privilege escalation risks—then implement automated guardrails to prevent and remediate these issues.

**Real-World Context:** IAM misconfigurations are the #1 cause of cloud security breaches. According to AWS security reports, 65% of cloud incidents involve compromised credentials or excessive permissions. This lab teaches you to prevent both.

---

## 🎯 Learning Objectives

By completing this scenario, you will demonstrate:

✅ **IAM Policy Analysis** - Identify overly permissive wildcard policies (`*`)  
✅ **Least Privilege Enforcement** - Apply principle of least privilege  
✅ **Credential Lifecycle Management** - Detect and rotate stale access keys  
✅ **MFA Enforcement** - Implement multi-factor authentication requirements  
✅ **Privilege Escalation Detection** - Identify dangerous permission combinations  
✅ **Automated Remediation** - Use Lambda to auto-fix IAM violations  
✅ **Compliance Mapping** - Align with CIS AWS Foundations Benchmark

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    IAM Misconfiguration Flow                 │
└─────────────────────────────────────────────────────────────┘

  1. Create Risky IAM Policy
        │
        ▼
  2. CloudTrail Logs API Call
        │
        ▼
  3. EventBridge Rule Triggered
        │
        ▼
  4. Lambda: iam_policy_analyzer()
        │
        ├──► Analyze Policy Document
        ├──► Check for Wildcards (Action: *, Resource: *)
        ├──► Check for Admin Access (AdministratorAccess)
        ├──► Check for Privilege Escalation Combos
        │
        ▼
  5. Risk Assessment
        │
        ├──► LOW: Log warning
        ├──► MEDIUM: SNS alert to security team
        └──► HIGH/CRITICAL: Auto-quarantine + immediate alert
        │
        ▼
  6. CloudWatch Dashboard Updated
        │
        ▼
  7. Compliance Report Generated
```

---

## 📚 Prerequisites

### Knowledge Requirements
- Basic understanding of AWS IAM (users, roles, policies)
- Familiarity with JSON policy documents
- AWS CLI experience

### Technical Requirements
- AWS account with admin permissions
- AWS CLI configured (`aws configure`)
- Python 3.9+ installed
- Terraform deployed (from main lab setup)

### Cost
**Estimated:** $0.00 (100% within free tier)
- CloudTrail: First trail free
- Lambda: 1M requests/month free
- CloudWatch: 10 metrics free
- SNS: 1,000 notifications free

---

## 🚀 Scenario Walkthrough

### **Part 1: Wildcard Permission Policy (CRITICAL)**

**Vulnerability:** Policy grants `*:*` on `Resource: *` (full admin access to everything)

#### Step 1: Create the Vulnerable Policy

```bash
# Create overly permissive policy document
cat > wildcard-policy.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DangerousWildcard",
      "Effect": "Allow",
      "Action": "*",
      "Resource": "*"
    }
  ]
}
EOF

# Create the policy in AWS
aws iam create-policy \
  --policy-name OverlyPermissive-DevPolicy \
  --policy-document file://wildcard-policy.json \
  --description "INTENTIONAL MISCONFIGURATION - Portfolio Demo"
```

#### Step 2: Monitor Detection

```bash
# Watch CloudWatch Logs for detection (real-time)
aws logs tail /aws/lambda/iam-guardrail-analyzer --follow --format short

# Expected output within 30-60 seconds:
# [IAM_CRITICAL] Wildcard policy detected: OverlyPermissive-DevPolicy
# [ACTION] Quarantine flag applied, security team notified
```

#### Step 3: Check SNS Alert

**Expected Email Alert:**
```
🚨 CRITICAL IAM VIOLATION DETECTED 🚨

Policy Name: OverlyPermissive-DevPolicy
Risk Level: CRITICAL
Issue: Full wildcard permissions (*:* on *)

Details:
- Grants unrestricted access to ALL AWS services
- Equivalent to AdministratorAccess
- Violates least privilege principle
- CIS AWS Benchmark: FAIL (1.16)

Automated Actions Taken:
✓ Policy flagged for review
✓ Security team notified
✓ Compliance report updated

Recommended Remediation:
1. Delete this policy immediately
2. Create service-specific policies
3. Review who created this policy
4. Implement policy approval workflow

Created By: arn:aws:iam::123456789012:user/charles-bucher
Created At: 2025-01-02T15:23:45Z
```

#### Step 4: Verify Guardrail Response

```bash
# Check if policy was quarantined (tagged for deletion)
aws iam get-policy --policy-arn arn:aws:iam::ACCOUNT_ID:policy/OverlyPermissive-DevPolicy

# Look for quarantine tag in output:
# Tags: [{"Key": "SecurityStatus", "Value": "QUARANTINED"}]

# View compliance dashboard
aws cloudwatch get-dashboard --dashboard-name iam-security-dashboard
```

---

### **Part 2: S3 Wildcard on All Resources (HIGH)**

**Vulnerability:** Policy grants `s3:*` on `*` (full S3 access to all buckets)

#### Step 1: Create S3 Wildcard Policy

```bash
cat > s3-wildcard-policy.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:*",
      "Resource": "*"
    }
  ]
}
EOF

aws iam create-policy \
  --policy-name S3-AllBuckets-FullAccess \
  --policy-document file://s3-wildcard-policy.json
```

#### Step 2: Expected Detection

```json
{
  "EventType": "IAM_POLICY_VIOLATION",
  "Severity": "HIGH",
  "PolicyName": "S3-AllBuckets-FullAccess",
  "Violation": "S3WildcardOnAllResources",
  "Impact": "User can access/delete ANY S3 bucket in the account",
  "Recommendation": "Restrict to specific bucket ARNs: arn:aws:s3:::my-bucket/*"
}
```

---

### **Part 3: Privilege Escalation Risk (HIGH)**

**Vulnerability:** Combination of permissions allows privilege escalation

#### Step 1: Create Privilege Escalation Policy

```bash
cat > privesc-policy.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "iam:CreateAccessKey",
        "iam:CreateUser",
        "iam:AttachUserPolicy"
      ],
      "Resource": "*"
    }
  ]
}
EOF

aws iam create-policy \
  --policy-name IAM-UserManagement-Policy \
  --policy-document file://privesc-policy.json
```

#### Step 2: Why This Is Dangerous

**Attack Scenario:**
1. Attacker has this policy attached
2. Attacker creates new IAM user: `aws iam create-user --user-name backdoor-admin`
3. Attacker attaches AdministratorAccess: `aws iam attach-user-policy --user-name backdoor-admin --policy-arn arn:aws:iam::aws:policy/AdministratorAccess`
4. Attacker creates access keys: `aws iam create-access-key --user-name backdoor-admin`
5. Attacker now has persistent admin access

#### Step 3: Guardrail Detection

```bash
# Lambda detects dangerous permission combo
aws logs filter-log-events \
  --log-group-name /aws/lambda/iam-guardrail-analyzer \
  --filter-pattern "PrivilegeEscalation"

# Expected output:
# [IAM_HIGH] Privilege escalation risk detected
# [COMBO] CreateUser + AttachUserPolicy + CreateAccessKey
# [RISK] User can grant themselves admin privileges
```

---

### **Part 4: Stale Access Keys (MEDIUM)**

**Vulnerability:** Access keys older than 90 days (compliance violation)

#### Step 1: Create Test User with Access Key

```bash
# Create test user
aws iam create-user --user-name test-stale-key-user

# Create access key
aws iam create-access-key --user-name test-stale-key-user

# Simulate old key (for testing, manually set last-used date in DynamoDB)
```

#### Step 2: Run Access Key Audit

```bash
# Lambda runs daily audit
python lambda/iam_access_key_audit.py

# Check results
aws dynamodb scan --table-name iam-access-key-inventory
```

#### Step 3: Expected Output

```json
{
  "UserName": "test-stale-key-user",
  "AccessKeyId": "AKIAIOSFODNN7EXAMPLE",
  "CreateDate": "2024-10-01T10:00:00Z",
  "LastUsedDate": "2024-10-05T12:30:00Z",
  "DaysSinceLastUse": 88,
  "Status": "WARN",
  "Action": "RotationRecommended",
  "ComplianceStatus": "CIS_1.4_FAIL"
}
```

---

### **Part 5: Missing MFA on IAM Users (HIGH)**

**Vulnerability:** IAM users without MFA enabled

#### Step 1: Create User Without MFA

```bash
aws iam create-user --user-name no-mfa-user

# Create password for console access
aws iam create-login-profile \
  --user-name no-mfa-user \
  --password 'TempPassword123!' \
  --password-reset-required
```

#### Step 2: Detection

```bash
# Lambda checks all users for MFA status
aws logs filter-log-events \
  --log-group-name /aws/lambda/iam-guardrail-analyzer \
  --filter-pattern "MFANotEnabled"
```

#### Step 3: Expected Alert

```
⚠️ IAM Security Alert: MFA Not Enabled

User: no-mfa-user
Risk Level: HIGH
Issue: Console access enabled without MFA

Impact:
- Account vulnerable to password compromise
- Violates CIS AWS Benchmark 1.2
- Fails SOC 2 control requirements

Required Action:
1. Enable MFA: aws iam enable-mfa-device
2. If not corrected in 24 hours, console access will be disabled
3. User will be notified via email

Compliance:
- CIS AWS Benchmark 1.2: FAIL
- NIST 800-53 IA-2(1): NON-COMPLIANT
```

---

## 🔍 Validation & Testing

### Test 1: Verify Policy Detection Works

```bash
# Check Lambda function logs for all detections
aws logs tail /aws/lambda/iam-guardrail-analyzer --since 1h

# Count total violations detected
aws logs filter-log-events \
  --log-group-name /aws/lambda/iam-guardrail-analyzer \
  --filter-pattern "[timestamp, request_id, level=IAM_CRITICAL || level=IAM_HIGH]" \
  --start-time $(date -d '1 hour ago' +%s)000
```

### Test 2: Validate Email Alerts

```bash
# List SNS messages sent
aws sns list-subscriptions-by-topic \
  --topic-arn arn:aws:sns:us-east-1:ACCOUNT_ID:iam-security-alerts

# Expected: 5 emails received (one per misconfiguration created)
```

### Test 3: Check Compliance Dashboard

```bash
# Generate compliance report
python scripts/generate_iam_compliance_report.py

# View summary
cat reports/iam-compliance-$(date +%Y-%m-%d).json | jq '.summary'
```

**Expected Output:**
```json
{
  "total_policies_scanned": 23,
  "violations_found": 5,
  "critical": 1,
  "high": 3,
  "medium": 1,
  "low": 0,
  "compliance_score": "78%",
  "cis_benchmark_status": "FAIL"
}
```

---

## 🛡️ Remediation Steps

### Manual Remediation (Learning Exercise)

#### Fix 1: Replace Wildcard Policy
```bash
# Delete dangerous policy
aws iam delete-policy \
  --policy-arn arn:aws:iam::ACCOUNT_ID:policy/OverlyPermissive-DevPolicy

# Create least privilege policy
cat > s3-readonly-specific.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::my-specific-bucket",
        "arn:aws:s3:::my-specific-bucket/*"
      ]
    }
  ]
}
EOF

aws iam create-policy \
  --policy-name S3-MyBucket-ReadOnly \
  --policy-document file://s3-readonly-specific.json
```

#### Fix 2: Rotate Stale Access Keys
```bash
# List old access keys
aws iam list-access-keys --user-name test-stale-key-user

# Create new key
aws iam create-access-key --user-name test-stale-key-user

# (After updating applications with new key)
# Delete old key
aws iam delete-access-key \
  --user-name test-stale-key-user \
  --access-key-id AKIAIOSFODNN7EXAMPLE
```

#### Fix 3: Enable MFA
```bash
# Enable virtual MFA device
aws iam enable-mfa-device \
  --user-name no-mfa-user \
  --serial-number arn:aws:iam::ACCOUNT_ID:mfa/no-mfa-user \
  --authentication-code-1 123456 \
  --authentication-code-2 789012
```

---

## 📊 Expected Results

### Metrics to Track

| Metric | Before | After Remediation |
|--------|---------|-------------------|
| Total IAM Policies | 23 | 23 |
| Wildcard Policies | 5 | 0 |
| Users Without MFA | 3 | 0 |
| Stale Access Keys (>90 days) | 2 | 0 |
| Privilege Escalation Risks | 2 | 0 |
| **CIS Benchmark Compliance** | **68%** | **100%** |

### CloudWatch Alarms Triggered
- ✅ Critical IAM policy created (1 alarm)
- ✅ High-risk S3 wildcard detected (1 alarm)
- ✅ Privilege escalation combo detected (1 alarm)
- ✅ MFA compliance check failed (1 alarm)
- ✅ Access key rotation overdue (1 alarm)

---

## 🧹 Cleanup

After completing the scenario, remove test resources:

```bash
# Delete test policies
aws iam delete-policy --policy-arn arn:aws:iam::ACCOUNT_ID:policy/OverlyPermissive-DevPolicy
aws iam delete-policy --policy-arn arn:aws:iam::ACCOUNT_ID:policy/S3-AllBuckets-FullAccess
aws iam delete-policy --policy-arn arn:aws:iam::ACCOUNT_ID:policy/IAM-UserManagement-Policy

# Delete test users
aws iam delete-login-profile --user-name no-mfa-user
aws iam delete-user --user-name no-mfa-user
aws iam delete-access-key --user-name test-stale-key-user --access-key-id AKIA...
aws iam delete-user --user-name test-stale-key-user

# Verify cleanup
aws iam list-policies --scope Local | grep -E "OverlyPermissive|AllBuckets|UserManagement"
# Should return no results
```

---

## 💼 Interview Talking Points

Use these key points when discussing this scenario in interviews:

### Technical Skills Demonstrated
1. **IAM Policy Analysis:** "I analyzed JSON policy documents to identify wildcards, privilege escalation risks, and excessive permissions."
2. **Automated Detection:** "I implemented EventBridge rules to trigger Lambda functions that analyze CloudTrail events in real-time."
3. **Security Best Practices:** "I enforced MFA, implemented least privilege access, and automated access key rotation."
4. **Compliance Mapping:** "I mapped technical controls to CIS AWS Foundations Benchmark and NIST 800-53."

### Real-World Application
*"In this lab, I simulated 5 common IAM misconfigurations I might encounter in production. The wildcard permission policy is especially dangerous because it essentially gives admin access. I built automated guardrails that detect these within 30 seconds and alert the security team. This demonstrates my understanding of both offensive and defensive cloud security."*

### Problem-Solving Example
*"When I discovered the privilege escalation risk, I had to think like an attacker. Someone with CreateUser, AttachUserPolicy, and CreateAccessKey can basically create their own backdoor admin account. My Lambda function now flags this specific combination and quarantines policies that contain it."*

---

## 📚 Additional Resources

### CIS AWS Foundations Benchmark Controls Covered
- **1.2** - Ensure multi-factor authentication (MFA) is enabled
- **1.4** - Ensure access keys are rotated every 90 days
- **1.16** - Ensure IAM policies are attached only to groups or roles
- **1.22** - Ensure IAM policies that allow full "*:*" are not created

### AWS Security Best Practices
- [IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [Least Privilege Principle](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html#grant-least-privilege)
- [IAM Policy Evaluation Logic](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html)

---

## 🎓 Learning Outcomes

After completing this scenario, you can confidently say:

✅ "I can analyze IAM policies for security vulnerabilities"  
✅ "I implemented automated IAM compliance checks"  
✅ "I understand privilege escalation attack vectors"  
✅ "I can map technical controls to compliance frameworks"  
✅ "I built event-driven security remediation workflows"

---

## 🔗 Related Scenarios

- **[002-S3-Public-Access](../002-s3-public-access/)** - S3 bucket security
- **[003-Encryption-Enforcement](../003-encryption-enforcement/)** - Data encryption
- **[004-Security-Group-Hardening](../004-security-group-hardening/)** - Network security

---

**Next Steps:** Complete [Scenario 002: S3 Public Access Detection](../002-s3-public-access/) to continue building your cloud security portfolio.

---

*Built by Charles Bucher | [GitHub](https://github.com/charles-bucher) | [LinkedIn](https://linkedin.com/in/charles-bucher-cloud)*
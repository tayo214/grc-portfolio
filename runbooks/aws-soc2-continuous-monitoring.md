# AWS SOC 2 Continuous Monitoring Runbook

**Author:** Babatunde Salako  
**Role:** Cloud Security Engineer  
**Certification:** CompTIA Security+  
**Version:** 1.0  
**Last Reviewed:** March 2026  
**Classification:** Internal — GRC Portfolio

---

## Purpose

This runbook documents the operational procedures for maintaining continuous controls monitoring (CCM) of an AWS environment aligned to SOC 2 Trust Service Criteria (TSC). It is designed to be executed by a security or GRC practitioner responsible for day-to-day compliance operations in a cloud-native environment.

The goal of continuous monitoring is to ensure that security controls remain effective between audit periods, that findings are identified and remediated on defined timelines, and that audit evidence is collected systematically rather than reactively during audit preparation.

This runbook covers the AWS-native toolchain used in a production environment: Security Hub, CloudTrail, GuardDuty, Amazon Inspector, AWS Config, and CloudWatch. It maps each tool to the SOC 2 Trust Service Criteria it supports and defines the cadence, procedure, and evidence artifact for each monitoring activity.

---

## Scope

- **Environment:** AWS Production Account
- **Frameworks:** SOC 2 Trust Service Criteria (2017 AICPA), CIS AWS Foundations Benchmark v3.0, AWS Foundational Security Best Practices (FSBP)
- **Tools Covered:** AWS Security Hub, CloudTrail, GuardDuty, Amazon Inspector, AWS Config, CloudWatch Logs, Splunk
- **Out of Scope:** On-premises systems, SaaS platforms, development and staging environments

---

## SOC 2 TSC to AWS Tool Mapping

| TSC Category | Criteria | Primary AWS Tool | Supporting Tool |
|---|---|---|---|
| Logical Access | CC6.1 – CC6.8 | IAM / Security Hub | CloudTrail, CloudWatch |
| Change Management | CC8.1 | AWS Config | CloudTrail |
| Threat Detection | CC7.1 – CC7.5 | GuardDuty | Security Hub, Splunk |
| Risk Monitoring | CC4.1 – CC4.2 | Security Hub | Wiz, Nessus |
| Availability | A1.1 – A1.3 | CloudWatch | Config, RDS Multi-AZ |
| Confidentiality | C1.1 – C1.2 | Security Hub | KMS, Inspector |
| Vulnerability Management | CC7.1 | Amazon Inspector | Wiz, Nessus |

---

## Monitoring Cadence Overview

| Activity | Frequency | SOC 2 TSC | Evidence Artifact |
|---|---|---|---|
| Security Hub findings review | Daily | CC4.1, CC7.2 | Findings export / screenshot |
| GuardDuty findings triage | Daily | CC7.2, CC7.3 | Findings triage log |
| CloudTrail anomaly review | Daily | CC7.2, CC8.1 | CloudWatch alarm log |
| IAM access review | Monthly | CC6.1, CC6.3 | IAM credential report |
| Inspector vulnerability review | Weekly | CC7.1 | Inspector findings report |
| AWS Config compliance check | Weekly | CC8.1 | Config compliance snapshot |
| Security Hub standards review | Monthly | CC4.1 | Security Hub score report |
| Privileged access audit | Monthly | CC6.2 | CloudTrail + IAM audit log |
| Full controls evidence collection | Quarterly | All TSC | Evidence package folder |
| Tabletop exercise | Semi-annual | CC9.2 | Exercise summary and AAR |

---

## Procedure 1 — Daily: Security Hub Findings Review

**TSC:** CC4.1, CC7.2  
**Tool:** AWS Security Hub  
**Time Required:** 15 – 30 minutes

### Steps

1. Navigate to AWS Security Hub in the AWS Management Console.
2. Select **Findings** from the left navigation panel.
3. Filter findings by **Record State = Active** and **Workflow Status = New**.
4. Sort by **Severity** (Critical first, then High).
5. For each Critical or High finding:
   - Review the finding title, affected resource, and remediation guidance.
   - Assign a workflow status: **Notified** (if routed to a team), **In Progress** (if being remediated), or **Suppressed** (if risk accepted with documented rationale).
   - Log the finding ID, severity, resource, and action taken in the findings triage log.
6. Confirm that no Critical findings have exceeded the defined SLA (Critical: 24 hours, High: 7 days, Medium: 30 days).
7. Export an updated findings summary for the evidence record.

### Evidence Artifact
- Daily findings triage log entry (finding ID, severity, resource ARN, action, timestamp)
- Weekly: exported CSV of active findings from Security Hub

### Key Standards Active
- AWS Foundational Security Best Practices (FSBP)
- CIS AWS Foundations Benchmark v3.0

---

## Procedure 2 — Daily: GuardDuty Findings Triage

**TSC:** CC7.2, CC7.3  
**Tool:** Amazon GuardDuty, Splunk  
**Time Required:** 10 – 20 minutes

### Steps

1. Navigate to Amazon GuardDuty in the AWS Console.
2. Select **Findings** and filter by **Severity: High and Critical**.
3. For each High or Critical finding:
   - Review the finding type (e.g., UnauthorizedAccess, CryptoCurrency, Trojan).
   - Identify the affected resource (EC2 instance, IAM principal, S3 bucket).
   - Cross-reference with Splunk for correlated log activity around the same time window.
   - Determine if the finding represents a true positive, false positive, or expected behavior.
   - Document disposition in the triage log.
4. For true positives: escalate per the Incident Response Plan. Assign severity, notify the ISSO, and open a tracking ticket.
5. For false positives: suppress the finding with a documented rationale. Do not suppress without written justification.
6. Confirm GuardDuty is active in all regions via the GuardDuty settings page.

### Evidence Artifact
- GuardDuty findings triage log (finding ID, type, resource, disposition, timestamp)
- Splunk correlation query results for High/Critical findings

### Common Finding Types to Watch
| Finding Type | Likely Cause | Response |
|---|---|---|
| UnauthorizedAccess:IAMUser/ConsoleLoginSuccess | Unusual login geography or time | Verify with user, check MFA |
| Recon:EC2/PortProbeUnprotectedPort | External port scanning | Review security group rules |
| CryptoCurrency:EC2/BitcoinTool | Cryptomining activity | Isolate instance immediately |
| Stealth:IAMUser/CloudTrailLoggingDisabled | CloudTrail tampered | Treat as incident, escalate |

---

## Procedure 3 — Daily: CloudTrail Anomaly Review

**TSC:** CC7.2, CC8.1  
**Tool:** AWS CloudTrail, CloudWatch  
**Time Required:** 10 minutes

### Steps

1. Navigate to CloudWatch in the AWS Console and select **Alarms**.
2. Review the status of all security-relevant metric filter alarms:
   - Root account login
   - Unauthorized API calls
   - IAM policy changes
   - Security Group rule changes
   - CloudTrail configuration changes
3. For any alarm in **ALARM** state:
   - Click through to the CloudWatch Logs Insights query to identify the triggering event.
   - Identify the IAM principal, source IP, and affected resource.
   - Determine if the action was authorized. Cross-check with change management records.
   - Log the event, determination, and any follow-up action.
4. Confirm CloudTrail is delivering logs to S3 without interruption (check S3 bucket for latest log delivery timestamp).
5. Verify log file integrity validation is active (CloudTrail console > Trail settings > Log file validation = Enabled).

### Evidence Artifact
- CloudWatch alarm state log
- CloudTrail event record for any investigated activity (exported JSON)

### Metric Filters Reference
| Alarm Name | Filter Pattern | SNS Topic |
|---|---|---|
| RootAccountUsage | `{ $.userIdentity.type = "Root" }` | security-alerts |
| UnauthorizedAPICalls | `{ $.errorCode = "AccessDenied" }` | security-alerts |
| IAMPolicyChanges | `{ $.eventName = "PutUserPolicy" OR ... }` | security-alerts |
| SecurityGroupChanges | `{ $.eventName = "AuthorizeSecurityGroupIngress" }` | security-alerts |
| CloudTrailChanges | `{ $.eventName = "StopLogging" }` | security-alerts |

---

## Procedure 4 — Weekly: Inspector Vulnerability Review

**TSC:** CC7.1  
**Tool:** Amazon Inspector  
**Time Required:** 20 – 30 minutes

### Steps

1. Navigate to Amazon Inspector in the AWS Console.
2. Select **Findings** and filter by **Severity: Critical and High**.
3. Review findings for EC2 instances and ECR container images separately.
4. For each Critical finding:
   - Identify the affected resource (instance ID or ECR image digest).
   - Review the CVE ID, CVSS score, and recommended remediation.
   - Confirm whether a patch is available. If yes, escalate to the infrastructure team with the defined SLA (Critical: 7 days, High: 30 days).
   - Log the CVE, affected resource, patch availability, assigned owner, and target remediation date.
5. For ECR findings: confirm that image scanning on push is enabled for all private repositories. Verify that critical image findings are blocking deployment where pipeline integration is configured.
6. Cross-reference EC2 findings with SSM Patch Manager compliance report to confirm patch baseline status.

### Evidence Artifact
- Inspector findings export (weekly CSV filtered to Critical/High)
- SSM Patch Manager compliance report screenshot

---

## Procedure 5 — Weekly: AWS Config Compliance Check

**TSC:** CC8.1  
**Tool:** AWS Config  
**Time Required:** 15 minutes

### Steps

1. Navigate to AWS Config in the AWS Console.
2. Select **Rules** and filter by **Compliance Status = Non-compliant**.
3. For each non-compliant rule:
   - Identify the rule name, affected resource, and the configuration that caused non-compliance.
   - Determine if the non-compliance is a known exception (documented, risk-accepted) or a new deviation.
   - For new deviations: open a remediation task with the infrastructure team and log in the findings tracker.
4. Review the **Configuration Timeline** for any resources that changed state during the past week.
5. Confirm that Config is recording in all regions and delivering snapshots to S3.

### Evidence Artifact
- Config compliance summary screenshot (non-compliant rule count by week)
- Config snapshot delivery confirmation

### Key Config Rules to Monitor
| Config Rule | SOC 2 TSC | What It Checks |
|---|---|---|
| iam-root-access-key-check | CC6.1 | No root access keys active |
| mfa-enabled-for-iam-console-access | CC6.1 | MFA on all console users |
| s3-bucket-public-read-prohibited | C1.1 | No public S3 read access |
| encrypted-volumes | C1.1 | EBS volumes encrypted |
| cloudtrail-enabled | CC7.2 | CloudTrail active in all regions |
| ec2-imdsv2-check | C1.2 | IMDSv2 enforced on EC2 |
| rds-storage-encrypted | C1.1 | RDS encryption at rest |
| restricted-ssh | CC6.6 | No unrestricted SSH (port 22) |

---

## Procedure 6 — Monthly: IAM Access Review

**TSC:** CC6.1, CC6.3  
**Tool:** AWS IAM  
**Time Required:** 30 – 45 minutes

### Steps

1. Generate an IAM Credential Report from the IAM console (Users > Credential Report > Download).
2. Review the report for:
   - Users with console access but no MFA enabled — flag and escalate for immediate remediation.
   - Access keys older than 90 days — notify the key owner and request rotation.
   - Access keys that have never been used or not used in 90+ days — flag for deactivation.
   - Users who have not logged in for 90+ days — flag for review with their manager.
3. Review IAM roles for any with `*:*` or wildcard administrative policies (Security Hub: IAM.1).
4. Confirm no IAM policies are attached directly to users — policies must be attached to groups or roles (Security Hub: IAM.16).
5. Document findings in the access review log. Route exceptions for sign-off per the access control policy.

### Evidence Artifact
- IAM Credential Report (dated, stored in evidence folder)
- Access review log with findings and disposition

---

## Procedure 7 — Quarterly: Evidence Collection Package

**TSC:** All  
**Tool:** All  
**Time Required:** 2 – 4 hours

### Purpose
Quarterly evidence collection ensures that audit-ready artifacts are maintained continuously and are not assembled under pressure during an audit. Each quarter, a complete evidence package is compiled covering all active SOC 2 controls.

### Evidence Checklist

| Evidence Item | Source | SOC 2 TSC |
|---|---|---|
| Security Hub findings summary (Active, past 90 days) | Security Hub export | CC4.1, CC7.2 |
| GuardDuty findings triage log | Internal log | CC7.2, CC7.3 |
| CloudTrail configuration screenshot (multi-region, validation on) | Console screenshot | CC7.2, CC8.1 |
| IAM Credential Report | IAM console | CC6.1, CC6.3 |
| IAM policy review results (no direct attachments, no wildcard admin) | IAM audit | CC6.3 |
| Inspector findings report (Critical/High) | Inspector export | CC7.1 |
| Config compliance summary (non-compliant count by rule) | Config console | CC8.1 |
| SSM Patch Manager compliance report | SSM console | CC7.1 |
| MFA status confirmation (root + all console users) | IAM + Security Hub | CC6.1 |
| CloudWatch alarm status (all security alarms active) | CloudWatch screenshot | CC7.2 |
| Incident log (any incidents in period, with disposition) | IR log | CC7.3 |
| Architecture diagram (current) | ISSO artifact | CC6.6 |
| Incident Response Plan (current version + review date) | Policy repository | CC9.2 |
| Disaster Recovery Plan (current version + last test date) | Policy repository | A1.2 |

### Evidence Folder Structure
```
evidence/
  YYYY-QN/
    security-hub-findings-summary.csv
    guardduty-triage-log.xlsx
    cloudtrail-config-screenshot.png
    iam-credential-report.csv
    iam-policy-review-log.xlsx
    inspector-findings-report.csv
    config-compliance-summary.png
    ssm-patch-compliance-report.png
    cloudwatch-alarms-status.png
    incident-log.xlsx
    architecture-diagram.pdf
    irp-current-version.pdf
    drp-current-version.pdf
```

---

## Remediation SLA Reference

| Severity | Remediation Target | Escalation Path |
|---|---|---|
| Critical | 24 hours | Security Lead + ISSO immediately |
| High | 7 days | Security Lead within 48 hours |
| Medium | 30 days | GRC / Security Analyst |
| Low | 90 days | Tracked in findings log |
| Informational | Next review cycle | No escalation required |

---

## Roles and Responsibilities

| Role | Responsibility |
|---|---|
| Cloud Security Engineer | Execute daily, weekly, and monthly monitoring procedures; maintain findings log |
| ISSO | Review escalated findings; sign off on risk acceptance; maintain policy artifacts |
| Infrastructure / DevOps | Remediate findings routed to them within defined SLA |
| Security Lead / CISO | Approve risk acceptance for Critical and High findings; review quarterly evidence package |

---

## Related Documents

- Incident Response Plan (IRP) — maintained by ISSO
- Disaster Recovery Plan — maintained by Infrastructure team
- Access Control Policy — maintained by Security team
- SOC 2 Controls Mapping — `controls-mapping/soc2-cis-fsbp-mapping.xlsx`
- Vendor Risk Assessment Template — `vendor-risk/vendor-risk-assessment-template.xlsx`

---

## Document Control

| Version | Date | Author | Change Summary |
|---|---|---|---|
| 1.0 | March 2026 | Babatunde Salako | Initial version |
# AI-Assisted GRC Workflow

**Author:** Babatunde Salako  
**Role:** Cloud Security Engineer  
**Certification:** CompTIA Security+  
**Version:** 1.0  
**Last Reviewed:** March 2026  
**Classification:** Internal — GRC Portfolio

---

## Purpose

This document demonstrates how AI tools — specifically Claude (Anthropic) and ChatGPT (OpenAI) — can be integrated into day-to-day GRC operations to reduce manual effort, improve response quality, and increase throughput without sacrificing accuracy or control.

The workflows documented here are practical and repeatable. Each one identifies the GRC task, the problem it solves, the AI-assisted approach, a worked example, and the human review step that ensures accuracy and accountability. AI is treated as a force multiplier for the GRC practitioner, not a replacement for judgment.

These workflows are directly applicable to the three highest-volume, most time-intensive functions in a customer-facing GRC program: security questionnaire response, controls mapping, and audit evidence summarization.

---

## Why AI in GRC

Modern GRC programs — particularly at high-growth companies — face a volume problem. Security questionnaires arrive from partners and customers faster than a single analyst can respond to them thoroughly. Controls need to be mapped across multiple frameworks simultaneously. Audit evidence needs to be summarized and organized under time pressure.

The traditional approach — handling each task manually from scratch — does not scale. The result is either slow turnaround that delays business deals, or rushed responses that introduce inaccuracy and compliance risk.

AI tools address this by handling the first draft, the structural work, and the pattern recognition — leaving the GRC practitioner to focus on accuracy review, customization, and the judgment calls that require domain expertise and organizational context.

The key discipline is that AI output is always reviewed, verified against source documentation, and approved before it leaves the GRC function. The workflows below make that review step explicit.

---

## Workflow 1 — Security Questionnaire Response Acceleration

### Problem
Inbound security questionnaires from customers and partners are time-intensive. A single SIG Lite or CAIQ can contain 50 to 150 questions. Responding thoroughly and consistently to multiple questionnaires per month — each with slightly different phrasing for the same underlying controls — consumes a disproportionate amount of GRC capacity.

### AI-Assisted Approach
Use Claude to generate a first-draft response to each questionnaire question based on a structured prompt that includes the organization's control environment as context. The GRC practitioner reviews each response, adjusts for accuracy against current documentation, and approves before submission.

### Prompt Template

```
You are a GRC analyst responding to a security questionnaire on behalf of a cloud-native 
organization. The organization's security environment is as follows:

- Cloud platform: AWS (production)
- Active compliance standards: CIS AWS Foundations Benchmark v3.0, AWS Foundational 
  Security Best Practices (FSBP)
- Key tools: AWS Security Hub, GuardDuty, CloudTrail, Amazon Inspector, AWS Config, 
  CloudWatch, Wiz, Nessus, Splunk
- Certifications: SOC 2 Type 2 (in progress), CompTIA Security+
- Controls in place: MFA enforced for all IAM users, least-privilege IAM, EBS/RDS/S3 
  encryption at rest, TLS in transit, IMDSv2 enforced, multi-region CloudTrail with 
  log validation, Security Hub findings reviewed daily

Respond to the following questionnaire question with a clear, professional, and accurate 
answer. Do not fabricate controls that are not listed above. If a control is partially 
implemented, say so. Keep the response to 3–5 sentences.

Question: [PASTE QUESTION HERE]
```

### Worked Example

**Questionnaire Question:**  
*Does your organization have a documented process for managing and responding to security vulnerabilities, including defined remediation timelines?*

**AI-Generated First Draft (Claude):**  
Yes. The organization operates a formal vulnerability management program using Amazon Inspector for continuous EC2 and ECR scanning, Wiz for cloud security posture management, and Nessus for infrastructure-level scanning. Findings are triaged by severity with defined remediation SLAs: Critical vulnerabilities are remediated within 24 hours, High within 7 days, and Medium within 30 days. Security Hub aggregates findings across tools for centralized review. Patch compliance is enforced through AWS Systems Manager Patch Manager and monitored on a weekly basis.

**Human Review Checklist:**
- Confirm IMDSv2 and patch timelines match current policy documentation
- Verify SLA figures are consistent with the Incident Response Plan
- Remove or qualify any detail that does not reflect current state
- Adjust tone to match organizational voice if needed

**Time Saved:** Estimated 20–40 minutes per question for complex controls questions, when starting from a blank page. Across a 100-question questionnaire, this can reduce total response time by 30–50%.

### Knowledge Base Approach
For organizations using a GRC platform like Vanta or Drata, the same prompt template can be used to pre-populate a questionnaire response knowledge base. Approved AI-generated responses are reviewed, finalized, and stored against specific question categories. Future questionnaires map incoming questions to the knowledge base for instant retrieval, with AI used only for new or edge-case questions.

---

## Workflow 2 — Cross-Framework Controls Mapping

### Problem
Mapping controls across multiple frameworks — for example, mapping existing CIS AWS controls to SOC 2 TSC, then identifying gaps against ISO 27001 — is analytical work that requires reading and comparing large volumes of framework documentation. Done manually, a full cross-framework mapping exercise can take days.

### AI-Assisted Approach
Use Claude to perform the initial mapping pass for a batch of controls, then review and verify each mapping against the authoritative framework documentation. This is particularly effective for identifying which TSC criteria a given AWS control satisfies, and for flagging where a control partially satisfies multiple criteria.

### Prompt Template

```
You are a GRC analyst performing a controls mapping exercise. Map the following AWS 
security control to the most relevant SOC 2 Trust Service Criteria category and 
specific criteria ID. Provide a one-sentence rationale for the mapping. If the control 
maps to multiple criteria, list all relevant ones.

AWS Control: [CONTROL TITLE AND DESCRIPTION]
Source Framework: [CIS AWS Foundations Benchmark v3.0 / AWS FSBP]

Return your response in this format:
- TSC Category: [Category name]
- Criteria ID(s): [e.g., CC6.1, CC6.3]
- Rationale: [One sentence]
```

### Worked Example

**Input Control:**  
*CIS 1.10 — Ensure MFA is enabled for all IAM users with console access (AWS FSBP: IAM.5)*

**AI-Generated Mapping (Claude):**
- TSC Category: Logical Access
- Criteria ID(s): CC6.1, CC6.2
- Rationale: MFA for console users directly supports CC6.1 (logical access security measures, including authentication) and CC6.2 (authentication mechanisms that restrict access to authorized users), as it adds a required second factor before granting access to production systems.

**Human Review Step:**  
Verify CC6.1 and CC6.2 language in the AICPA TSC documentation. Confirm no additional criteria apply (e.g., CC6.8 for access restriction to prevent unauthorized actions). Approve and add to the controls mapping workbook.

### Gap Analysis Extension
After completing the initial mapping, use a follow-up prompt to identify gaps:

```
Based on the following SOC 2 TSC criteria list, identify which criteria are NOT 
addressed by the AWS controls mapped below. For each gap, suggest what type of 
control or evidence would satisfy the criteria.

Mapped controls: [PASTE MAPPING TABLE]
TSC criteria list: CC6.1, CC6.2, CC6.3, CC6.6, CC6.7, CC6.8, CC7.1, CC7.2, CC7.3, 
CC8.1, A1.1, A1.2, C1.1, C1.2
```

---

## Workflow 3 — Audit Evidence Summarization

### Problem
During audit preparation, the GRC practitioner must compile and summarize evidence across dozens of controls. Raw evidence — Security Hub exports, CloudTrail logs, IAM credential reports — needs to be translated into clear, auditor-readable summaries that map each artifact to the control it satisfies. Doing this from scratch for every audit cycle is repetitive and time-consuming.

### AI-Assisted Approach
Use Claude to generate a structured evidence summary from a description of the raw artifacts collected. The practitioner provides a description of what was collected and from where; Claude formats it into an auditor-readable summary tied to SOC 2 criteria. The practitioner verifies the summary against the actual artifacts before finalizing.

### Prompt Template

```
You are a GRC analyst preparing audit evidence documentation for a SOC 2 Type 2 audit. 
Based on the following evidence artifacts collected for the audit period, write a 
structured evidence summary for each control. The summary should be written in the 
third person, reference the specific artifact, and identify the SOC 2 TSC criteria 
it satisfies. Keep each summary to 2–3 sentences.

Control: [CONTROL NAME]
TSC Criteria: [e.g., CC6.1]
Evidence Collected: [DESCRIBE ARTIFACT — e.g., "IAM Credential Report dated 
March 1, 2026 showing MFA enabled for all 47 IAM users with console access. 
Screenshot of Security Hub finding IAM.5 showing 0 non-compliant resources."]
```

### Worked Example

**Input:**  
Control: MFA Enforcement for IAM Console Users  
TSC Criteria: CC6.1  
Evidence Collected: IAM Credential Report dated March 1, 2026 — all 47 IAM users with console access have MFA enabled. Security Hub finding IAM.5 shows 0 non-compliant resources as of March 1, 2026.

**AI-Generated Evidence Summary (Claude):**  
The organization enforces multi-factor authentication for all IAM users with AWS Management Console access. An IAM Credential Report dated March 1, 2026 confirms that all 47 users with console access have MFA enabled, with no exceptions. This is corroborated by AWS Security Hub finding IAM.5, which shows zero non-compliant resources as of the same date, satisfying the continuous monitoring requirement under CC6.1.

**Human Review Step:**  
Confirm the user count (47) matches the current IAM Credential Report. Verify the Security Hub screenshot date aligns with the evidence period. Confirm no exceptions or suppressions exist for IAM.5 that would undermine the summary.

---

## Workflow 4 — Vendor Security Questionnaire Outbound Review

### Problem
When onboarding new vendors, the GRC team must review vendor-completed questionnaires and identify gaps or unsatisfactory responses. Reading through a completed questionnaire and extracting a gap summary manually is time-consuming and can be inconsistent across reviewers.

### AI-Assisted Approach
Paste the vendor's completed questionnaire responses into Claude with a structured prompt to extract gaps, inconsistencies, and areas requiring follow-up documentation.

### Prompt Template

```
You are a GRC analyst reviewing a completed vendor security questionnaire. Read the 
following vendor responses and identify:
1. Any responses that indicate a control is not implemented or only partially implemented
2. Any responses that are vague, inconsistent, or lack sufficient detail
3. Questions where additional evidence should be requested before approving the vendor

Format your output as a numbered gap list with the question reference, the issue, 
and a recommended follow-up action.

Vendor responses: [PASTE VENDOR QUESTIONNAIRE RESPONSES]
```

### Output Use
The gap list generated by Claude is reviewed by the GRC analyst, refined based on context and risk tier, and used as the basis for a vendor follow-up communication requesting additional evidence or clarification. It also populates the Risk Register in the Vendor Risk Assessment Template.

---

## Human Review Discipline

Across all four workflows, the following non-negotiable rules apply:

**AI output is a first draft, not a final answer.** Every AI-generated response, mapping, summary, or gap list must be reviewed by the GRC practitioner before it is used, shared, or stored as an official record.

**Source documentation is always authoritative.** If AI output conflicts with the organization's actual policy, controls documentation, or framework text, the source documentation wins. AI does not override what is written in the IRP, the access control policy, or the AICPA TSC criteria.

**Do not use AI for regulatory determination.** AI tools should not be used to determine whether a specific control satisfies a regulatory requirement for audit purposes without verification by a qualified reviewer. Framework mapping produced by AI is a starting point for human analysis, not a final compliance determination.

**Sensitive data stays out of prompts.** Personally identifiable information, financial data, internal system architecture details beyond what is necessary for the task, and access credentials must never be included in prompts to external AI tools. If working with sensitive evidence, describe it generically in the prompt rather than pasting it verbatim.

**Log AI-assisted outputs.** Maintain a log of which GRC artifacts were produced with AI assistance, the prompt used, and the reviewer who approved the final version. This supports auditability and ensures the organization can demonstrate that human review was applied.

---

## Tools Referenced

| Tool | Use in GRC Workflows | Access Model |
|---|---|---|
| Claude (Anthropic) | Questionnaire drafting, controls mapping, evidence summarization, gap analysis | claude.ai / API |
| ChatGPT (OpenAI) | Secondary drafting, cross-check of AI-generated mappings | chat.openai.com / API |
| Vanta / Drata | GRC platform — knowledge base storage, evidence collection automation, questionnaire management | SaaS |
| SafeBase | Trust center — customer-facing security documentation portal | SaaS |

---

## Document Control

| Version | Date | Author | Change Summary |
|---|---|---|---|
| 1.0 | March 2026 | Babatunde Salako | Initial version
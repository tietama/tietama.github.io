# TryHackMe - Placeholder

_"Dive into the heat of a live phishing attack as it unfolds within the corporate network. In this high-pressure scenario, your role is to meticulously analyse and document each phase of the breach as it happens._

_Can you piece together the attack chain in real-time and prepare a comprehensive report on the malicious activities?"_

## Executive Summary

This case study documents a simulated Security Operations Center (SOC) investigation in which multiple low-severity alerts initially appeared to be routine false positives. Further analysis revealed that one alert represented the beginning of a targeted phishing attack against a company executive.

The incident progressed from malicious email delivery to endpoint compromise, internal reconnaissance, access to sensitive financial records, and attempted data exfiltration within minutes.

This exercise demonstrated the importance of effective alert triage, contextual risk assessment, and rapid escalation when weak signals may indicate high-impact threats.

---

## Incident Objective

- Monitor and analyse real-time alerts as the attack unfolds.
- Identify and document critical events such as PowerShell executions, reverse shell connections, and suspicious DNS requests.
- Create detailed case reports based on your observations to help the team understand the full scope of the breach.

---

# Phase 1 – Early Alert Noise

The scenario began with several low-severity alerts involving suspicious inbound emails and uncommon Windows process parent-child relationships.

Examples included:

- inbound messages from unusual top-level domains  
- generic spam / phishing themes  
- legitimate Windows processes flagged due to what appeared to be rarity-based detection logic

These alerts were reviewed but did not initially show evidence of compromise.

![Placeholder – suspicious email alert screenshot](assets/images/early-alerts.png)

### Analyst Assessment

These detections appeared to reflect tuning gaps rather than active malicious activity.

### Key Observation

A high alert count does not necessarily equal high risk.

---

# Phase 2 – First Meaningful Signal

A new alert identified a suspicious email attachment delivered directly to the company CEO.

**Subject:**

`FINAL NOTICE: Overdue Payment - Account Suspension Imminent`

**Attachment:**

`ImportantInvoice-Febrary.zip`

Although still labeled low severity, this alert required immediate reprioritization because:

- malicious attachments present execution risk  
- single-recipient delivery suggested targeting  
- executive compromise could create significant business impact

![Placeholder – suspicious attachment alert screenshot](assets/images/attachment-alert.png)

### Analyst Assessment

This was the first alert that warranted urgent escalation.

### Key Lesson

Severity labels should not override business context.

---

# Phase 3 – Confirmed Endpoint Compromise

Subsequent telemetry showed the attachment was opened via Windows Explorer, triggering PowerShell execution.

Observed command activity included:

- in-memory script download via `IEX(...)`
- retrieval of **PowerCat**
- reverse shell creation to external infrastructure

Example behavior:

- remote PowerShell command execution  
- outbound command-and-control activity

![Placeholder – PowerShell execution screenshot](assets/images/powershell-execution.png)

### Tooling Observed

**PowerCat** – PowerShell-based networking utility commonly abused for reverse shells and command execution.

### Analyst Assessment

This confirmed successful user execution and attacker foothold on the endpoint.

---

# Phase 4 – Reconnaissance & Internal Discovery

Following initial compromise, logs showed system and user enumeration activity.

Examples included:

- `systeminfo.exe`
- `whoami.exe`

Additional tooling indicated use of:

**PowerView** – PowerShell framework commonly used for Active Directory and internal network reconnaissance.

A PowerView script was also created in the user Downloads directory.

![Placeholder – reconnaissance screenshot](assets/images/reconnaissance.png)

### Analyst Assessment

The attacker was actively gathering host and domain intelligence to identify follow-on opportunities.

---

# Phase 5 – Collection of Sensitive Data

The attacker then mapped an internal network share:

`\\FILESRV-01\SSF-FinancialRecords`

Drive `Z:` was mounted temporarily.

Shortly afterward, logs showed **Robocopy** creating files in a local exfiltration staging folder.

Observed file:

`ClientPortfolioSummary.xlsx`

![Placeholder – network share screenshot](assets/images/share-access.png)

### Tooling Observed

**Robocopy** – legitimate Windows file copy utility often abused for bulk data staging and transfer.

### Analyst Assessment

This represented likely collection of sensitive business data prior to exfiltration.

---

# Phase 6 – Attempted Exfiltration

Final logs showed suspicious `nslookup.exe` execution with a long encoded-looking subdomain:

`UEsDBBQAAAA...haz4rdw4re.io`

This behavior is consistent with DNS-based data exfiltration or tunneling.

![Placeholder – DNS exfil screenshot](assets/images/dns-exfiltration.png)

### Analyst Assessment

The attacker likely attempted to move staged data externally through DNS requests.

---

# Attack Timeline Summary

1. Low-quality alerts created background noise  
2. Targeted phishing email delivered to CEO  
3. Malicious attachment opened  
4. PowerShell reverse shell established  
5. Reconnaissance performed  
6. Financial share accessed  
7. Files staged locally  
8. DNS exfiltration attempted

---

# Key Findings

## Detection Strengths

- Attachment detection successfully identified the initial delivery vector  
- Endpoint telemetry enabled full attack reconstruction

## Detection Gaps

- Several low-value alerts created noise before the real incident  
- Important alerts were labeled low severity despite elevated context

## Business Risk Factors

- Executive account targeted  
- Sensitive financial records accessed  
- Rapid attacker progression after execution

---

# Containment Opportunities

The strongest prevention window occurred between:

**Suspicious attachment alert generated**  
and  
**Attachment opened by the user**

Rapid analyst triage during that interval could potentially have enabled:

- user notification  
- attachment quarantine  
- sender blocking  
- host monitoring escalation

Once execution began, attacker activity progressed within minutes.

---

# Recommendations

## Detection Engineering

- Increase priority for suspicious attachments sent to executives  
- Reduce noise from weak TLD-only phishing detections  
- Tune parent-child rarity alerts with allowlists

## Response Readiness

- Accelerate triage workflows for high-risk recipients  
- Improve phishing reporting channels for executives  
- Automate host isolation for confirmed malicious PowerShell activity

## User Awareness

- Reinforce training around ZIP/LNK disguised attachments  
- Promote urgent reporting of suspicious invoice emails

---

# Skills Demonstrated

- Alert triage  
- False positive assessment  
- Timeline reconstruction  
- Log correlation  
- Windows process analysis  
- Incident reporting  
- Business-context prioritization

---

# Final Reflection

This scenario highlighted a common SOC challenge: genuine incidents are not always preceded by high-severity alerts. In many environments, the real skill lies in identifying which low-confidence signal deserves immediate attention.

The most dangerous alert in this case did not look the loudest at first glance.

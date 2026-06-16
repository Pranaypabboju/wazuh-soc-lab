# Incident Report — [INCIDENT TITLE]

> **How to use this template:** Fill in each section after investigating an alert in Wazuh.  
> Replace all `[PLACEHOLDER]` values with your findings.  
> Delete this instruction block before saving the final report.

---

## Incident Summary

| Field | Details |
|---|---|
| **Incident ID** | INC-[NUMBER] (e.g., INC-001) |
| **Date & Time Detected** | [YYYY-MM-DD HH:MM UTC] |
| **Analyst Name** | [YOUR NAME] |
| **Severity** | [ ] Critical  [ ] High  [ ] Medium  [ ] Low |
| **Status** | [ ] Open  [ ] In Progress  [ ] Resolved |
| **Affected Host** | [HOSTNAME / IP ADDRESS] |
| **Source IP (Attacker)** | [IP ADDRESS] |

---

## 1. Alert Details

**Wazuh Rule ID:** [e.g., 5763]  
**Rule Description:** [e.g., sshd: brute force trying to get access to the system]  
**Alert Severity Level:** [Wazuh level 0–15]  
**First Seen:** [timestamp]  
**Last Seen:** [timestamp]  
**Total Events:** [number of related events]

---

## 2. Description of Activity

*Write 2–4 sentences describing what happened in plain language. What did the attacker do? What system was targeted? What was observed?*

[DESCRIPTION]

**Example:**  
> Between 14:20 and 14:35 UTC, a large number of failed SSH login attempts were detected against the Ubuntu endpoint (192.168.56.30). The attempts originated from 192.168.56.20 (Kali Linux attacker machine) and targeted the root account. Wazuh Rule 5763 fired after the 8th failed attempt within a 2-minute window, escalating the alert to Critical severity. No successful logins were observed.

---

## 3. Timeline of Events

| Timestamp | Event | Source | Rule |
|---|---|---|---|
| [HH:MM:SS] | [What happened] | [Log source] | [Rule ID] |
| [HH:MM:SS] | [What happened] | [Log source] | [Rule ID] |
| [HH:MM:SS] | [What happened] | [Log source] | [Rule ID] |

---

## 4. Indicators of Compromise (IOCs)

*List all suspicious IPs, usernames, file hashes, or other artifacts found.*

| Type | Value | Context |
|---|---|---|
| IP Address | [x.x.x.x] | Source of attack |
| Username | [e.g., root] | Account targeted |
| Port | [e.g., 22] | Service targeted |
| User Agent | [if applicable] | - |

---

## 5. Log Evidence

*Paste the key raw log entries that support your findings. Include at least 3–5 relevant lines.*

```
[PASTE RAW LOG LINES HERE]

Example:
Dec 10 14:23:11 endpoint sshd[2891]: Failed password for root from 192.168.56.20 port 54321 ssh2
Dec 10 14:23:12 endpoint sshd[2892]: Failed password for root from 192.168.56.20 port 54322 ssh2
Dec 10 14:23:41 endpoint sshd[2901]: PAM 8 more authentication failures; user=root
```

---

## 6. MITRE ATT&CK Mapping

| ATT&CK Element | Value |
|---|---|
| **Tactic** | [e.g., Credential Access] |
| **Technique ID** | [e.g., T1110.001] |
| **Technique Name** | [e.g., Brute Force: Password Guessing] |
| **Link** | https://attack.mitre.org/techniques/[TECHNIQUE_ID]/ |

---

## 7. Impact Assessment

**Was the attack successful?** [ ] Yes  [ ] No  [ ] Unknown

**Affected Systems:** [List any systems impacted]

**Data at Risk:** [Any sensitive data that could have been exposed?]

**Business Impact:** [Describe potential impact if in a real environment]

---

## 8. Findings Summary

*Summarize your key findings in 3–5 bullet points.*

- [Finding 1]
- [Finding 2]
- [Finding 3]

---

## 9. Recommended Remediation Actions

*List specific, actionable steps to fix or mitigate the issue. Prioritize by urgency.*

### Immediate Actions (within 24 hours)
- [ ] [Action 1 — e.g., Block attacker IP via UFW]
- [ ] [Action 2 — e.g., Reset compromised credentials if login succeeded]

### Short-Term Actions (within 1 week)
- [ ] [Action 3 — e.g., Disable root SSH login]
- [ ] [Action 4 — e.g., Implement SSH key-based authentication]

### Long-Term Recommendations
- [ ] [Action 5 — e.g., Deploy fail2ban for automated IP blocking]
- [ ] [Action 6 — e.g., Set up MFA for remote access]

---

## 10. Lessons Learned

*What did this incident teach you? What would you do differently next time?*

[NOTES]

---

*Report prepared by: [YOUR NAME] | Date: [YYYY-MM-DD]*

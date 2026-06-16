# Incident Report — SSH Brute Force Attack Detection

> Investigation of unauthorized SSH login attempts against Ubuntu endpoint

---

## Incident Summary

| Field | Details |
|---|---|
| **Incident ID** | INC-001 |
| **Date & Time Detected** | 2024-12-10 14:23 UTC |
| **Analyst Name** | pranay pabboju |
| **Severity** | Critical |
| **Status** | Resolved |
| **Affected Host** | ubuntu-endpoint / 192.168.56.30 |
| **Source IP (Attacker)** | 192.168.42.128 |

---

## 1. Alert Details

**Wazuh Rule ID:** 5763  
**Rule Description:** sshd: brute force trying to get access to the system  
**Alert Severity Level:** 15 (Critical)  
**First Seen:** 2024-12-10 14:23:11 UTC  
**Last Seen:** 2024-12-10 14:35:42 UTC  
**Total Events:** 47 failed login attempts

---

## 2. Description of Activity

Between 14:23 and 14:35 UTC on December 10, 2024, a large-scale SSH brute force attack was detected against the Ubuntu endpoint (192.168.56.30). The attack originated from Kali Linux machine (192.168.42.128) and targeted the root user account. Approximately 47 failed SSH login attempts were recorded within a 12-minute window. Wazuh Rule 5763 triggered after the 8th failed attempt, automatically escalating the alert to Critical severity. The attack was simulated as part of a controlled SOC lab exercise and no successful compromise occurred.

---

## 3. Timeline of Events

| Timestamp | Event | Source | Rule |
|---|---|---|---|
| 14:23:11 | Failed password for root | /var/log/auth.log | 5710 |
| 14:23:12 | Failed password for root | /var/log/auth.log | 5710 |
| 14:23:13 | Failed password for root | /var/log/auth.log | 5710 |
| 14:23:14 | Failed password for root | /var/log/auth.log | 5710 |
| 14:23:15 | Failed password for root | /var/log/auth.log | 5710 |
| 14:23:16 | Failed password for root | /var/log/auth.log | 5710 |
| 14:23:17 | Failed password for root | /var/log/auth.log | 5710 |
| 14:23:18 | Failed password for root | /var/log/auth.log | 5710 |
| 14:23:19 | POSSIBLE BREAK-IN ATTEMPT | /var/log/auth.log | 5712 |
| 14:23:41 | PAM 8 more authentication failures; user=root | /var/log/auth.log | 5763 |

---

## 4. Indicators of Compromise (IOCs)

| Type | Value | Context |
|---|---|---|
| IP Address | 192.168.42.128 | Source of brute force attack |
| Username | root | Account targeted by attack |
| Port | 22 | SSH service attacked |
| Attack Tool | Hydra | Password guessing tool used |
| Time Range | 14:23-14:35 UTC | Attack window |

---

## 5. Log Evidence

```
Dec 10 14:23:11 endpoint sshd[2891]: Failed password for root from 192.168.56.20 port 54321 ssh2
Dec 10 14:23:12 endpoint sshd[2892]: Failed password for root from 192.168.56.20 port 54322 ssh2
Dec 10 14:23:13 endpoint sshd[2893]: Failed password for root from 192.168.56.20 port 54323 ssh2
Dec 10 14:23:14 endpoint sshd[2894]: Failed password for root from 192.168.56.20 port 54324 ssh2
Dec 10 14:23:15 endpoint sshd[2895]: Failed password for root from 192.168.56.20 port 54325 ssh2
Dec 10 14:23:16 endpoint sshd[2896]: Failed password for root from 192.168.56.20 port 54326 ssh2
Dec 10 14:23:17 endpoint sshd[2897]: Failed password for root from 192.168.56.20 port 54327 ssh2
Dec 10 14:23:18 endpoint sshd[2898]: Failed password for root from 192.168.56.20 port 54328 ssh2
Dec 10 14:23:41 endpoint sshd[2899]: PAM 8 more authentication failures; logname= uid=0 euid=0 tty=ssh ruser= rhost=192.168.56.20 user=root
```

---

## 6. MITRE ATT&CK Mapping

| ATT&CK Element | Value |
|---|---|
| **Tactic** | Credential Access |
| **Technique ID** | T1110.001 |
| **Technique Name** | Brute Force: Password Guessing |
| **Link** | https://attack.mitre.org/techniques/T1110/001/ |

---

## 7. Impact Assessment

**Was the attack successful?** No

**Affected Systems:** Ubuntu endpoint (192.168.56.30) — monitoring only, no compromise

**Data at Risk:** None — attack was blocked at SSH authentication stage

**Business Impact:** This was a simulated attack in a lab environment. In a production environment, such an attack could result in unauthorized access if the attacker guessed the correct password.

---

## 8. Findings Summary

- SSH brute force attack detected from 192.168.42.128 targeting root account
- 47 failed login attempts were recorded in a 12-minute window
- Wazuh SIEM successfully detected the attack using Rule 5763
- No successful authentication occurred; attack was contained
- Attack pattern matches MITRE ATT&CK technique T1110.001 (Password Guessing)

---

## 9. Recommended Remediation Actions

### Immediate Actions (within 24 hours)
- [x] Block attacker IP via UFW: `sudo ufw deny from 192.168.42.128`
- [x] Review all successful SSH logins in the past 24 hours for unauthorized access
- [x] Reset root password to a strong, unique value

### Short-Term Actions (within 1 week)
- [x] Disable root SSH login in `/etc/ssh/sshd_config` (set `PermitRootLogin no`)
- [x] Implement SSH key-based authentication instead of password-based
- [x] Change SSH from default port 22 to non-standard port (e.g., 2222)

### Long-Term Recommendations
- [x] Deploy fail2ban to automatically block IPs after failed attempts
- [x] Implement MFA (Multi-Factor Authentication) for remote access
- [x] Set up network IDS (Intrusion Detection System) for deeper packet inspection
- [x] Enable SSH logging and monitoring for all connection attempts

---

## 10. Lessons Learned

This incident demonstrated the importance of:
1. **Real-time alert detection** — Wazuh detected the attack within seconds
2. **Automated escalation** — Alert severity increased automatically after threshold
3. **Pattern recognition** — Multiple failed logins from same IP = brute force
4. **Proactive defense** — Monitoring in real-time allows rapid response

In a production environment, this attack would be caught and blocked in real-time, preventing any compromise. The SOC team would immediately isolate the source IP and initiate investigation protocols.

---

*Report prepared by: pranay pabboju | Date: 2026-06-12*
```

| Analyst Name| *Pranay pabboju* |
| Affected Host IP | 192.168.56.30| 
| Source IP | 192.168.42.128 | 
| Date & Time | 2024-12-10 14:23 UTC | 12/15:13 |
| Incident ID | INC-001 | 




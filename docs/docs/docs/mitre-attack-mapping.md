# MITRE ATT&CK Technique Mapping

This document maps the attack activity observed in this lab to the [MITRE ATT&CK framework](https://attack.mitre.org/) — an industry-standard knowledge base of adversary tactics and techniques.

---

## What is MITRE ATT&CK?

MITRE ATT&CK is a globally recognized framework that categorizes the methods attackers use to compromise systems. It organizes techniques into **Tactics** (the "why" — the goal) and **Techniques** (the "how" — the specific method).

Using this framework helps SOC analysts:
- Describe threats in a common language
- Understand what stage of an attack they're seeing
- Prioritize detection and response

---

## Technique Mapping Table

| Observed Activity | ATT&CK Tactic | Technique ID | Technique Name | Wazuh Rule(s) |
|---|---|---|---|---|
| SSH brute-force login attempts from Kali | **Credential Access** | T1110.001 | Brute Force: Password Guessing | 5763, 5712 |
| Nmap port scan from Kali | **Reconnaissance** | T1046 | Network Service Discovery | 1002, 2101 |
| Multiple failed authentication events | **Credential Access** | T1110 | Brute Force | 5710, 5711 |
| Suspicious process execution on endpoint | **Execution** | T1059.004 | Unix Shell | 5301 |
| Sudo command usage | **Privilege Escalation** | T1548.003 | Sudo and Sudo Caching | 5402 |

---

## Detailed Breakdown

### T1110.001 — Brute Force: Password Guessing

**Tactic:** Credential Access  
**What happened:** Hydra was used from Kali Linux to attempt hundreds of SSH logins per minute using a wordlist.

**How it looked in the logs:**
```
Dec 10 14:23:11 endpoint sshd[2891]: Failed password for root from 192.168.56.20 port 54321 ssh2
Dec 10 14:23:11 endpoint sshd[2892]: Failed password for root from 192.168.56.20 port 54322 ssh2
Dec 10 14:23:11 endpoint sshd[2893]: Failed password for root from 192.168.56.20 port 54323 ssh2
```

**Key Indicators:**
- Same source IP attempting many logins in a short time
- `Failed password` repeated for the same username
- Wazuh escalates severity after threshold is crossed (default: 8 failures = brute force alert)

**Wazuh Rule Fired:** `5763 — sshd: brute force trying to get access to the system`

---

### T1046 — Network Service Discovery

**Tactic:** Reconnaissance  
**What happened:** Nmap was run from Kali to discover open ports and services on the endpoint.

**How it looked in the logs:**
```
Dec 10 14:10:02 endpoint kernel: [UFW BLOCK] IN=eth0 SRC=192.168.56.20 DST=192.168.56.30 
  PROTO=TCP DPT=22
Dec 10 14:10:02 endpoint kernel: [UFW BLOCK] IN=eth0 SRC=192.168.56.20 DST=192.168.56.30 
  PROTO=TCP DPT=80
```

**Key Indicators:**
- Sequential port connections from the same source
- Many different destination ports in rapid succession
- Connection attempts to closed/filtered ports

---

### T1548.003 — Sudo and Sudo Caching

**Tactic:** Privilege Escalation  
**What happened:** Sudo commands were executed on the endpoint (both legitimate and as part of simulation).

**How it looked in the logs:**
```
Dec 10 14:35:00 endpoint sudo: user : TTY=pts/0 ; PWD=/home/user ; 
  USER=root ; COMMAND=/usr/bin/cat /etc/shadow
```

**Key Indicators:**
- Unusual sudo commands for a standard user
- Accessing sensitive files like `/etc/shadow` or `/etc/passwd`
- Sudo at unusual hours

---

## ATT&CK Tactic Coverage in This Lab

```
Reconnaissance     ██████████  ✓ Covered (port scanning)
Initial Access     ░░░░░░░░░░  Not simulated
Execution          ████░░░░░░  ✓ Partial (process execution)
Persistence        ░░░░░░░░░░  Not simulated
Privilege Escalation ██████░░  ✓ Partial (sudo)
Credential Access  ██████████  ✓ Covered (brute force)
Discovery          ██████████  ✓ Covered (network scan)
Lateral Movement   ░░░░░░░░░░  Not simulated
```

---

## Resources

- [MITRE ATT&CK Matrix for Enterprise](https://attack.mitre.org/matrices/enterprise/)
- [Wazuh MITRE ATT&CK integration](https://documentation.wazuh.com/current/user-manual/ruleset/mitre.html)
- [ATT&CK Navigator](https://mitre-attack.github.io/attack-navigator/) — visualize your coverage

# Attack Simulation 02 — SSH Brute Force

**Tool Used:** Hydra  
**Target:** Ubuntu endpoint SSH (port 22)  
**MITRE ATT&CK:** T1110.001 — Brute Force: Password Guessing  
**Expected Wazuh Alert:** Rule 5763 (Critical)

---

## What is SSH Brute Force?

SSH (Secure Shell) is used to remotely access Linux servers. Attackers try SSH brute force by attempting thousands of username/password combinations rapidly, hoping to guess the correct credentials and gain access.

---

## Pre-Requisites

- Kali Linux VM running and able to reach the endpoint
- SSH service must be running on the endpoint: `sudo systemctl start ssh`
- Hydra installed on Kali (pre-installed by default)

---

## Running the Simulation

> ⚠️ **Only run this against your own lab machines. Never test against systems you don't own.**

### Step 1 — Verify SSH is reachable from Kali
```bash
# From Kali Linux
nmap -p 22 [ENDPOINT_IP]
# Should show: 22/tcp open ssh
```

### Step 2 — Run the Hydra Brute Force
```bash
# Basic brute force targeting root user
hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://[ENDPOINT_IP] -t 4 -V

# Explanation of flags:
# -l root        → try logging in as 'root'
# -P rockyou.txt → use this password wordlist
# -t 4           → use 4 parallel threads
# -V             → verbose output (shows each attempt)
```

### Step 3 — Observe the attack running
You'll see output like:
```
[ATTEMPT] target [ENDPOINT_IP] - login "root" - pass "123456" - 1 of 14344392
[ATTEMPT] target [ENDPOINT_IP] - login "root" - pass "password" - 2 of 14344392
[ATTEMPT] target [ENDPOINT_IP] - login "root" - pass "12345678" - 3 of 14344392
```

Stop the attack after a minute or two with `Ctrl+C` — you don't need to complete it.

---

## What to Look for in Wazuh

### In the Dashboard → Events:
Filter by your endpoint's IP and look for these events:

| Wazuh Rule ID | Rule Description | Severity |
|---|---|---|
| 5710 | sshd: Attempt to login using a denied user | Medium |
| 5711 | sshd: Attempt to login using a non-existent user | Medium |
| 5712 | sshd: POSSIBLE BREAK-IN ATTEMPT! | High |
| **5763** | **sshd: brute force trying to get access** | **Critical** |

Rule 5763 fires automatically once Wazuh detects **8 or more failed login attempts from the same IP within 2 minutes**.

### Raw Log Example (from `/var/log/auth.log`):
```
Dec 10 14:23:11 endpoint sshd[2891]: Failed password for root from 192.168.56.20 port 54321 ssh2
Dec 10 14:23:12 endpoint sshd[2892]: Failed password for root from 192.168.56.20 port 54322 ssh2
Dec 10 14:23:12 endpoint sshd[2893]: Failed password for root from 192.168.56.20 port 54323 ssh2
Dec 10 14:23:13 endpoint sshd[2894]: Failed password for root from 192.168.56.20 port 54324 ssh2
```

---

## Investigation Checklist

After the attack, practice investigating it in Wazuh:

- [ ] Find the brute-force alert (Rule 5763) in Events
- [ ] Note the **source IP** (should be Kali's IP)
- [ ] Count how many failed attempts occurred
- [ ] Check the **timestamp range** — how long did the attack last?
- [ ] Look at the **MITRE ATT&CK** tab in Wazuh — does it map to T1110?
- [ ] Fill out the [Incident Report Template](../reports/templates/incident-report-template.md)

---

## Remediation Recommendations

If this happened on a real system, recommended actions would include:

1. **Block the attacker's IP** using UFW: `sudo ufw deny from [ATTACKER_IP]`
2. **Disable root SSH login** — edit `/etc/ssh/sshd_config` → set `PermitRootLogin no`
3. **Implement SSH key authentication** instead of passwords
4. **Install fail2ban** to automatically ban IPs after failed attempts
5. **Change SSH to a non-standard port** (e.g., 2222 instead of 22) to reduce noise

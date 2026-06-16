# Attack Simulation 01 — Port Scanning

**Tool Used:** Nmap  
**Target:** Ubuntu endpoint  
**MITRE ATT&CK:** T1046 — Network Service Discovery  
**Expected Wazuh Alert:** Rule 1002 / 2101

---

## What is Port Scanning?

Before attacking a system, attackers typically perform **reconnaissance** — gathering information about the target. Port scanning is a technique used to discover:

- Which ports are open (what services are running)
- What software versions are running on those ports
- The operating system of the target

Tools like Nmap send packets to a range of ports and analyze the responses to build a picture of the target.

---

## Pre-Requisites

- Kali Linux VM running and able to ping the endpoint
- Nmap installed on Kali (pre-installed by default)

---

## Running the Simulation

> ⚠️ **Only run this against your own lab machines. Never scan systems you don't own.**

### Scan 1 — Basic TCP Scan (quick)
```bash
# From Kali Linux — scan common ports
nmap [ENDPOINT_IP]

# Example output:
# PORT   STATE SERVICE
# 22/tcp open  ssh
# 80/tcp open  http
```

### Scan 2 — Aggressive Scan (generates more alerts)
```bash
# OS detection + version detection + script scanning
nmap -A -sV [ENDPOINT_IP]

# Explanation of flags:
# -A   → aggressive mode (OS + version + traceroute)
# -sV  → version detection (identify service software)
```

### Scan 3 — Full Port Scan (very noisy)
```bash
# Scan ALL 65535 ports — this is very loud and will trigger more alerts
nmap -p- [ENDPOINT_IP]
```

---

## What to Look for in Wazuh

### In the Dashboard → Events:
Look for alerts triggered by scan traffic:

| Wazuh Rule ID | Description | Severity |
|---|---|---|
| 1002 | Unknown problem somewhere in the system | Medium |
| 2101 | Host-based anomaly detection event | Medium |
| 40111 | Multiple connections from same source | Medium |

> **Note:** Port scan detection in Wazuh is more visible when UFW (firewall) is enabled on the endpoint, as blocked connection attempts are logged to `/var/log/kern.log`.

### Enable UFW on Endpoint (to generate better logs)
```bash
# On the Ubuntu endpoint
sudo ufw enable
sudo ufw logging on
```

Then re-run the Nmap scan. You'll see kernel log entries like:
```
kernel: [UFW BLOCK] IN=eth0 SRC=192.168.56.20 DPT=80
kernel: [UFW BLOCK] IN=eth0 SRC=192.168.56.20 DPT=443
kernel: [UFW BLOCK] IN=eth0 SRC=192.168.56.20 DPT=8080
```

---

## Investigation Checklist

- [ ] Identify the source IP of the scan in Wazuh Events
- [ ] Determine which ports were probed
- [ ] Note the time and duration of the scanning activity
- [ ] Check if this IP appeared in other alerts (follow-up attacks)
- [ ] Map to MITRE ATT&CK T1046
- [ ] Document in the [Incident Report Template](../reports/templates/incident-report-template.md)

---

## Remediation Recommendations

1. **Enable and configure UFW** to block unnecessary ports
2. **Monitor for scanning patterns** — many connections to different ports from one IP
3. **Use Wazuh active response** to automatically block scanning IPs
4. **Close unnecessary open ports** — only expose services that are needed
5. **Consider a network IDS** (e.g., Suricata) for deeper packet inspection

# 🛡️ Security Operations & Threat Monitoring Lab — Wazuh SIEM

A hands-on home lab simulating a real-world Security Operations Center (SOC) using Wazuh SIEM. This project covers deploying the monitoring infrastructure, onboarding endpoint agents, simulating attacks, investigating alerts, and documenting findings — the full analyst workflow.

---

## 📌 Table of Contents

- [What This Project Is About](#what-this-project-is-about)
- [Lab Environment](#lab-environment)
- [Architecture Overview](#architecture-overview)
- [What I Built & Did](#what-i-built--did)
- [Attack Scenarios Simulated](#attack-scenarios-simulated)
- [Alert Investigation Workflow](#alert-investigation-workflow)
- [MITRE ATT&CK Mapping](#mitre-attck-mapping)
- [Folder Structure](#folder-structure)
- [How to Replicate This Lab](#how-to-replicate-this-lab)
- [Key Takeaways](#key-takeaways)
- [Screenshots](#screenshots)

---

## What This Project Is About

This lab was built to practice blue team skills in a safe, isolated environment. The goal was to:

- Set up a centralized SIEM (Security Information and Event Management) platform
- Collect and monitor security logs from an endpoint
- Launch real attack techniques from a Kali Linux machine
- Detect, investigate, and triage the resulting alerts
- Document everything like a real SOC analyst would

> **Who is this for?** Anyone learning cybersecurity, preparing for roles like SOC Analyst, or studying for certifications like CompTIA Security+, CySA+, or Blue Team Level 1 (BTL1).

---

## Lab Environment

| Component | Role | Details |
|---|---|---|
| **Ubuntu Server** | Wazuh SIEM Server | Hosts the Wazuh Manager, Indexer, and Dashboard |
| **Ubuntu Endpoint** | Monitored Machine | Runs the Wazuh Agent to send logs |
| **Kali Linux** | Attacker Machine | Used to simulate attacks against the endpoint |

All machines were run as virtual machines (VMs) on the same local network.

---

## Architecture Overview

```
┌──────────────────────────────────────────────────────────┐
│                      Home Lab Network                    │
│                                                          │
│   ┌─────────────┐        ┌──────────────────────────┐   │
│   │  Kali Linux │        │     Ubuntu Server         │   │
│   │  (Attacker) │        │   Wazuh SIEM Manager      │   │
│   │             │        │   Wazuh Indexer            │   │
│   │  - nmap     │        │   Wazuh Dashboard (Web UI)│   │
│   │  - hydra    │        └────────────┬─────────────┘   │
│   │  - metaspl. │                     │ receives logs    │
│   └──────┬──────┘                     │                  │
│          │  attacks                   │                  │
│          ▼                            │                  │
│   ┌──────────────┐   agent sends logs │                  │
│   │Ubuntu Endpt. │───────────────────►│                  │
│   │ Wazuh Agent  │                                       │
│   └──────────────┘                                       │
└──────────────────────────────────────────────────────────┘
```

---

## What I Built & Did

### 1. Deployed Wazuh SIEM
- Installed Wazuh Manager, Indexer, and Dashboard on Ubuntu Server
- Accessed the web-based Wazuh Dashboard to view and manage alerts
- Configured log ingestion pipelines for authentication and system events

### 2. Configured Endpoint Monitoring
- Installed and enrolled the **Wazuh Agent** on the Ubuntu endpoint
- Agent automatically collects:
  - **Authentication logs** — `/var/log/auth.log` (login attempts, sudo usage)
  - **System events** — process creation, file changes
  - **Security alerts** — policy violations, anomalies

### 3. Simulated Attacks from Kali Linux
Launched controlled attack techniques to generate real security events. See [Attack Scenarios](#attack-scenarios-simulated) below.

### 4. Investigated Alerts
- Triaged alerts by **severity level** (low / medium / high / critical)
- Examined **timestamps**, **source IPs**, **affected hosts**, and **event details**
- Identified patterns indicating brute-force, scanning, and suspicious behavior

### 5. Mapped to MITRE ATT&CK
- Identified relevant techniques from the MITRE ATT&CK framework for each attack
- Linked observed behavior to known adversary tactics

### 6. Created Incident Documentation
- Wrote structured incident reports with findings and remediation steps
- See [`/reports/templates/`](./reports/templates/) for the template used

---

## Attack Scenarios Simulated

| # | Attack Type | Tool Used | What It Generates |
|---|---|---|---|
| 1 | **Port Scanning** | `nmap` | Network scan alerts, probe detection |
| 2 | **SSH Brute Force** | `hydra` | Repeated failed login attempts (Rule 5763) |
| 3 | **Failed Login Attempts** | Manual / hydra | Authentication failure logs |
| 4 | **Suspicious Process Execution** | bash scripts | Anomalous process alerts |

### Example: SSH Brute Force with Hydra
```bash
# Run from Kali Linux — targets SSH on the endpoint
hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://[ENDPOINT_IP] -t 4
```
> ⚠️ Only run this in your own lab environment. Never against systems you don't own.

### Example: Port Scan with Nmap
```bash
# Aggressive scan to trigger detection rules
nmap -A -sV [ENDPOINT_IP]
```

---

## Alert Investigation Workflow

This is the step-by-step process used to investigate each alert in Wazuh:

```
1. RECEIVE ALERT
   └── Check severity level (Wazuh levels 0–15)
       └── Level 7–10 = Medium | Level 11–14 = High | Level 15 = Critical

2. IDENTIFY CONTEXT
   ├── What time did it happen? (timestamp)
   ├── Which host was affected? (agent.name / agent.ip)
   ├── What user was involved? (data.srcuser / data.dstuser)
   └── What was the source IP? (data.srcip)

3. ANALYSE THE EVENT
   ├── Read the full log entry
   ├── Check the Wazuh rule that fired (rule.id + rule.description)
   └── Look for patterns — is this a one-off or repeated?

4. DETERMINE INTENT
   ├── Brute Force? → Many failures in short time from same IP
   ├── Scanning? → Sequential port probes
   └── Suspicious Behaviour? → Unusual process / user activity

5. DOCUMENT & RESPOND
   ├── Fill in the Incident Report Template
   ├── Note affected assets, timeline, and IOCs
   └── Recommend remediation actions
```

---

## MITRE ATT&CK Mapping

| Observed Activity | MITRE Tactic | Technique ID | Technique Name |
|---|---|---|---|
| SSH brute-force login attempts | Credential Access | T1110.001 | Brute Force: Password Guessing |
| Port scanning from Kali | Reconnaissance | T1046 | Network Service Discovery |
| Failed authentication events | Defense Evasion | T1078 | Valid Accounts (attempted) |
| Suspicious process execution | Execution | T1059 | Command and Scripting Interpreter |

> Full ATT&CK mapping details are in [`/docs/mitre-attack-mapping.md`](./docs/mitre-attack-mapping.md)

---

## Folder Structure

```
wazuh-soc-lab/
│
├── README.md                        ← You are here
│
├── docs/
│   ├── lab-setup-guide.md           ← Step-by-step setup instructions
│   ├── wazuh-agent-config.md        ← How the agent was configured
│   └── mitre-attack-mapping.md      ← Detailed ATT&CK technique mapping
│
├── configs/
│   └── ossec.conf.example           ← Example Wazuh agent config (sanitized)
│
├── attack-simulations/
│   ├── 01-port-scan.md              ← Nmap scan scenario & expected alerts
│   └── 02-ssh-brute-force.md        ← Hydra brute force scenario & expected alerts
│
├── reports/
│   └── templates/
│       └── incident-report-template.md   ← Blank template for incident documentation
│
└── screenshots/
    └── (add your Wazuh dashboard screenshots here)
```

---

## How to Replicate This Lab

### Prerequisites
- A computer capable of running 3 VMs (8GB RAM minimum recommended)
- Virtualization software: [VirtualBox](https://www.virtualbox.org/) (free) or VMware
- ISO images: Ubuntu Server 22.04, Kali Linux

### Quick Setup Steps

1. **Create 3 VMs** on the same Host-Only or NAT network
2. **Install Wazuh on Ubuntu Server** — follow the [official Wazuh quickstart guide](https://documentation.wazuh.com/current/quickstart.html)
3. **Install Wazuh Agent on Ubuntu Endpoint** — see [`/docs/wazuh-agent-config.md`](./docs/wazuh-agent-config.md)
4. **Verify logs are flowing** in the Wazuh Dashboard → Agents section
5. **Run attack simulations** from Kali Linux — see [`/attack-simulations/`](./attack-simulations/)
6. **Investigate alerts** using the workflow above
7. **Document your findings** using the template in [`/reports/templates/`](./reports/templates/)

> Detailed step-by-step instructions are in [`/docs/lab-setup-guide.md`](./docs/lab-setup-guide.md)

---

## Key Takeaways

- **Hands-on SIEM experience** — learned how to navigate Wazuh, filter alerts, and read raw log data
- **Attacker perspective** — understanding *how* attacks happen makes detection much more intuitive
- **Analyst workflow** — practiced the full triage pipeline from raw alert to documented incident
- **MITRE ATT&CK** — learned to map real observed behavior to a standardized framework used by the industry
- **Documentation matters** — clear, structured incident reports are a core SOC skill

---

## Screenshots

> Add screenshots of your Wazuh Dashboard here to make this README more visual.
> Suggested screenshots to include:
> - Wazuh Dashboard overview (agents connected)
> - Alert list showing brute-force detections
> - Individual alert drill-down view
> - MITRE ATT&CK module view

Place `.png` or `.jpg` files in the `/screenshots/` folder and link them like:
```markdown
![Wazuh Dashboard](./screenshots/dashboard-overview.png)
```

---

## 📄 License

This project is for educational purposes. Feel free to fork and adapt for your own learning.

---

*Built as part of a cybersecurity home lab practice. Tools used: Wazuh SIEM, Ubuntu Server, Kali Linux.*

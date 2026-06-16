# Wazuh Agent Configuration

This document explains how the Wazuh Agent was configured on the monitored Ubuntu endpoint.

---

## What is the Wazuh Agent?

The Wazuh Agent is a lightweight program installed on the machine you want to monitor (the "endpoint"). It:

- Reads log files on that machine
- Monitors file integrity (detects changes to important files)
- Reports events back to the Wazuh Server in real time
- Runs local security checks (rootkit detection, vulnerability scanning)

Think of it as a "security sensor" sitting on the endpoint, sending everything it sees back to the central SIEM.

---

## Config File Location

The agent's main configuration file is located at:
```
/var/ossec/etc/ossec.conf
```

See [`/configs/ossec.conf.example`](../configs/ossec.conf.example) for the sanitized example used in this lab.

---

## Key Configuration Sections

### 1. Server Connection
Tells the agent where the Wazuh Manager (server) is:
```xml
<client>
  <server>
    <address>192.168.56.10</address>   <!-- Replace with your server IP -->
    <port>1514</port>
    <protocol>tcp</protocol>
  </server>
</client>
```

### 2. Log Monitoring
Defines which log files on the endpoint are collected:
```xml
<localfile>
  <log_format>syslog</log_format>
  <location>/var/log/auth.log</location>
</localfile>

<localfile>
  <log_format>syslog</log_format>
  <location>/var/log/syslog</location>
</localfile>

<localfile>
  <log_format>syslog</log_format>
  <location>/var/log/dpkg.log</location>
</localfile>
```

**Why these files?**
- `/var/log/auth.log` — records all login attempts, sudo usage, and SSH sessions. Critical for detecting brute-force attacks.
- `/var/log/syslog` — general system messages, useful for spotting unusual activity.
- `/var/log/dpkg.log` — package install/remove history. Attackers sometimes install tools after compromising a system.

### 3. File Integrity Monitoring (FIM)
Watches important directories and alerts if any files change:
```xml
<syscheck>
  <frequency>300</frequency>   <!-- Check every 5 minutes -->
  <directories check_all="yes">/etc,/usr/bin,/usr/sbin</directories>
  <directories check_all="yes">/bin,/sbin</directories>
</syscheck>
```

**Why?** If an attacker gains access, they often modify system files or add malicious scripts. FIM catches this.

### 4. Active Response (Optional)
Can automatically block IPs after too many failed logins:
```xml
<active-response>
  <disabled>no</disabled>
  <ca_store>etc/wpk_root.pem</ca_store>
</active-response>
```

> In this lab, active response was kept disabled to allow full observation of attacks without interference.

---

## Verifying Agent Status

Check if the agent is running:
```bash
sudo systemctl status wazuh-agent
```

Check the agent log for connection details:
```bash
sudo tail -f /var/ossec/logs/ossec.log
```

Look for lines like:
```
INFO: Connected to the server.
INFO: Sending keep alive message.
```

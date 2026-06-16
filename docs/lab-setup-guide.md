# Lab Setup Guide

This guide walks you through building the Wazuh SOC lab from scratch. Follow each section in order.

---

## Step 1 — Create Your Virtual Machines

You need **3 virtual machines** all connected on the same network.

### Recommended VM Settings

| VM | OS | RAM | Disk | Role |
|---|---|---|---|---|
| wazuh-server | Ubuntu Server 22.04 | 4 GB | 50 GB | Wazuh SIEM |
| endpoint | Ubuntu Server 22.04 | 2 GB | 20 GB | Monitored machine |
| attacker | Kali Linux | 2 GB | 20 GB | Attack simulation |

### Network Setup (VirtualBox)
1. Open VirtualBox → File → Host Network Manager → Create a new Host-Only network
2. Set each VM's network adapter to **Host-Only Adapter** using that network
3. This keeps all VMs isolated from the internet but able to talk to each other

---

## Step 2 — Install Wazuh on Ubuntu Server

Wazuh has an all-in-one installer that sets up the Manager, Indexer, and Dashboard together.

```bash
# Download the Wazuh installer
curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh

# Make it executable and run it
chmod +x wazuh-install.sh
sudo bash wazuh-install.sh -a
```

When installation completes, note down the **admin username and password** printed in the terminal.

### Access the Dashboard
Open a browser and go to:
```
https://[YOUR_WAZUH_SERVER_IP]
```
Log in with the credentials from the installer output.

> If you can't reach it, check that port 443 is open: `sudo ufw allow 443`

---

## Step 3 — Install Wazuh Agent on the Endpoint

Run these commands **on the Ubuntu endpoint machine** (not the server).

```bash
# Add the Wazuh repository
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo gpg --dearmor -o /usr/share/keyrings/wazuh.gpg
echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | sudo tee /etc/apt/sources.list.d/wazuh.list

# Install the agent
sudo apt update
sudo apt install wazuh-agent

# Register the agent with the Wazuh server
# Replace [WAZUH_SERVER_IP] with your server's IP address
sudo WAZUH_MANAGER='[WAZUH_SERVER_IP]' WAZUH_AGENT_NAME='ubuntu-endpoint' dpkg-reconfigure wazuh-agent

# Start and enable the agent
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
```

### Verify the Agent is Connected
In the Wazuh Dashboard → go to **Agents** → your endpoint should appear with status **Active**.

---

## Step 4 — Configure Log Monitoring

The agent config file controls what logs are collected. See [`/configs/ossec.conf.example`](../configs/ossec.conf.example) for the full example.

Key log sources configured on the endpoint:

```xml
<!-- Inside /var/ossec/etc/ossec.conf on the endpoint -->

<localfile>
  <log_format>syslog</log_format>
  <location>/var/log/auth.log</location>   <!-- SSH and sudo events -->
</localfile>

<localfile>
  <log_format>syslog</log_format>
  <location>/var/log/syslog</location>     <!-- General system events -->
</localfile>
```

After editing, restart the agent:
```bash
sudo systemctl restart wazuh-agent
```

---

## Step 5 — Verify Everything is Working

Before running any attacks, confirm the pipeline is healthy:

1. **Dashboard → Agents** — endpoint shows as Active
2. **Dashboard → Events** — you can see logs coming in from the endpoint
3. **Try triggering a test alert** — run a failed SSH login from Kali and see if it appears:
   ```bash
   # From Kali Linux
   ssh wronguser@[ENDPOINT_IP]
   # Enter any wrong password
   ```
4. Back in Wazuh Dashboard → **Events** → filter by your endpoint's IP → you should see the authentication failure

If you see the event, the lab is fully working. Move on to the attack simulations.

---

## Troubleshooting

| Problem | Solution |
|---|---|
| Agent shows Disconnected | Check firewall: `sudo ufw allow 1514/tcp` and `1515/tcp` on server |
| No events in Dashboard | Restart agent: `sudo systemctl restart wazuh-agent` |
| Can't reach Dashboard | Ensure port 443 open, try `https://` not `http://` |
| VMs can't ping each other | Check all VMs are on same Host-Only network in VirtualBox |

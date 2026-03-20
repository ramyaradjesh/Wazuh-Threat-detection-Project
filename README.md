# Wazuh Installation Guide

## Lab Environment
| Machine | OS | Role | IP |
|---|---|---|---|
| Ubuntu 1 | Ubuntu 22.04 | Wazuh Manager + Dashboard | 192.168.110.x |
| Ubuntu 2 | Ubuntu 22.04 | Wazuh Agent + DVWA + MariaDB | 192.168.110.x |
| Kali Linux | Kali 2024 | Attacker Simulation | 192.168.110.x |

---

## Part 1 — Wazuh Manager Installation (Ubuntu 1)

### Step 1: Update System
```bash
sudo apt update && sudo apt upgrade -y
```

### Step 2: Install Dependencies
```bash
sudo apt install curl apt-transport-https unzip wget -y
```

### Step 3: Add Wazuh Repository
```bash
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | \
sudo gpg --dearmor -o /usr/share/keyrings/wazuh.gpg

echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] \
https://packages.wazuh.com/4.x/apt/ stable main" | \
sudo tee /etc/apt/sources.list.d/wazuh.list

sudo apt update
```

### Step 4: Install Wazuh Manager
```bash
sudo apt install wazuh-manager -y
sudo systemctl enable wazuh-manager
sudo systemctl start wazuh-manager
sudo systemctl status wazuh-manager
```

### Step 5: Install Wazuh Indexer (OpenSearch)
```bash
sudo apt install wazuh-indexer -y
sudo systemctl enable wazuh-indexer
sudo systemctl start wazuh-indexer
```

### Step 6: Install Wazuh Dashboard
```bash
sudo apt install wazuh-dashboard -y
sudo systemctl enable wazuh-dashboard
sudo systemctl start wazuh-dashboard
```

### Step 7: Access Dashboard
```
URL: https://<Ubuntu-1-IP>:443
Default credentials:
  Username: admin
  Password: admin
```

---

## Part 2 — Wazuh Agent Installation (Ubuntu 2)

### Step 1: Add Wazuh Repository
```bash
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | \
sudo gpg --dearmor -o /usr/share/keyrings/wazuh.gpg

echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] \
https://packages.wazuh.com/4.x/apt/ stable main" | \
sudo tee /etc/apt/sources.list.d/wazuh.list

sudo apt update
```

### Step 2: Install Agent
```bash
sudo apt install wazuh-agent -y
```

### Step 3: Register Agent with Manager
```bash
sudo /var/ossec/bin/agent-auth \
  -m <Ubuntu-1-Manager-IP> \
  -A ubuntu2-agent
```

### Step 4: Configure Manager IP
```bash
sudo nano /var/ossec/etc/ossec.conf
```
Edit the `<server>` block:
```xml
<server>
  <address>192.168.110.x</address>
  <port>1514</port>
  <protocol>tcp</protocol>
</server>
```

### Step 5: Start Agent
```bash
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
sudo systemctl status wazuh-agent
```

### Step 6: Verify Agent on Dashboard
```
Wazuh Dashboard → Agents → Check ubuntu2-agent shows Active
```

---

## Part 3 — Apache Log Monitoring Setup (Ubuntu 2)

Add to `/var/ossec/etc/ossec.conf` inside `<ossec_config>`:
```xml
<localfile>
  <log_format>apache</log_format>
  <location>/var/log/apache2/access.log</location>
</localfile>

<localfile>
  <log_format>apache</log_format>
  <location>/var/log/apache2/error.log</location>
</localfile>
```

Restart agent:
```bash
sudo systemctl restart wazuh-agent
```

---

## Part 4 — File Integrity Monitoring (FIM) Setup

Add to `ossec.conf` on Ubuntu 2:
```xml
<syscheck>
  <directories check_all="yes" realtime="yes"
    report_changes="yes">/tmp</directories>
  <directories check_all="yes">/home</directories>
  <directories check_all="yes">/var/lib/mysql</directories>
  <directories check_all="yes">/etc</directories>
</syscheck>
```

---

## Part 5 — Verify Full Pipeline
```bash
# On Ubuntu 1 — check alerts are flowing
sudo tail -f /var/ossec/logs/alerts/alerts.log

# Search for specific rule hits
sudo grep "100501" /var/ossec/logs/alerts/alerts.json
```

Expected flow:
```
Agent (Ubuntu 2) → Manager (Ubuntu 1) → Rule Engine → Alerts → Dashboard
```

---

## Troubleshooting

| Issue | Fix |
|-------|-----|
| Agent shows Disconnected | Check firewall — open port 1514/tcp on Manager |
| No alerts in dashboard | Restart both manager and agent |
| Dashboard unreachable | Check wazuh-dashboard service status |
| Rules not triggering | Run `sudo /var/ossec/bin/wazuh-analysisd -t` to validate XML |

---

## References
- Wazuh Official Docs: https://documentation.wazuh.com
- Wazuh Installation Assistant: https://wazuh.com/install
- MITRE ATT&CK: https://attack.mitre.org

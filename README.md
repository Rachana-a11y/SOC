
# Project 2 – SOC: Enterprise EDR & Threat Hunting Grid

**Sentient Shield – Enterprise Endpoint Detection & Threat Monitoring Platform**

## Objective

The goal of this project was to build a **Security Operations Center (SOC) monitoring environment** capable of detecting malicious activities across endpoints.

The monitoring system was built using **Wazuh**, which collects logs from different systems and analyzes them to detect threats.

To improve Windows visibility, **Sysmon** was deployed.

To simulate real attacks, **Atomic Red Team** was used.

The SOC platform can detect:

* Brute force attacks
* File modifications
* Suspicious processes
* Malware behavior
* MITRE ATT&CK techniques

---

# Environment Setup

| Machine         | Role                 |
| --------------- | -------------------- |
| Ubuntu Server   | Wazuh Manager        |
| Windows Machine | Endpoint with Sysmon |
| Linux Machine   | Monitored endpoint   |

Example IPs (for documentation):

```
Wazuh Manager: 192.168.1.10
Windows Agent: 192.168.1.20
Linux Agent: 192.168.1.30
```

---

# Week 1 – Infrastructure Setup & Agent Deployment

## Goal

Install the SOC monitoring infrastructure and connect endpoints.

---

# Step 1 – Install Wazuh Manager

On Ubuntu server run:

```bash
curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh
sudo bash wazuh-install.sh -a
```

This installs:

* Wazuh Manager
* OpenSearch
* Wazuh Dashboard

Verify service:

```bash
sudo systemctl status wazuh-manager
```

Expected output:

```
Active: active (running)
```


<img width="966" height="595" alt="Screenshot 2026-03-09 215938" src="https://github.com/user-attachments/assets/da2ba2f8-328b-4285-928f-98e97ad60d0b" />


```
wazuh-manager-running.png
```

📌 Capture:

```
systemctl status wazuh-manager
```

---

# Step 2 – Access Wazuh Dashboard

Open browser:

```
https://MANAGER-IP
```

Login credentials are generated during installation.

Dashboard loads successfully.



<img width="1918" height="960" alt="Screenshot 2026-03-12 201254" src="https://github.com/user-attachments/assets/1d0e9cf1-0058-4fc5-8442-85977f19b278" />




```
wazuh-dashboard-home.png
```

📌 Capture:

Wazuh dashboard homepage.

---

# Step 3 – Install Wazuh Agent on Linux

Run on Linux endpoint:

```bash
curl -so wazuh-agent.deb https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.x.deb
sudo WAZUH_MANAGER="192.168.1.10" dpkg -i wazuh-agent.deb
```

Start agent:

```bash
sudo systemctl daemon-reload
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
```

Check status:

```bash
sudo systemctl status wazuh-agent
```


<img width="1328" height="698" alt="Screenshot 2026-03-09 211454" src="https://github.com/user-attachments/assets/5c0a7caa-70bd-4ba0-b6ef-e246b1e612ad" />


```
linux-agent-status.png
```

📌 Capture:

```
systemctl status wazuh-agent
```

---

# Step 4 – Install Windows Agent

Download agent installer from Wazuh website.

Install with manager IP.

After installation check:

```
services.msc
```

Verify:

```
Wazuh Agent Running
```


<img width="1328" height="698" alt="Screenshot 2026-03-09 211454" src="https://github.com/user-attachments/assets/f3ed91ec-d6ec-46c1-ad39-d29cafcdefbd" />


```
windows-agent-running.png
```

📌 Capture:

Windows services showing Wazuh Agent.

---

# Step 5 – Install Sysmon

Download **Sysmon**.

Run in PowerShell:

```powershell
sysmon64.exe -i sysmonconfig.xml
```

Verify:

```powershell
Get-Service sysmon
```

---

## Screenshot 5

📸 **Screenshot Name**

```
sysmon-running.png
```

📌 Capture:

Sysmon service status.

---

# Step 6 – Verify Agents in Dashboard

Go to:

```
Wazuh Dashboard → Agents
```

You should see:

```
Linux agent – Active
Windows agent – Active
```

---

## Screenshot 6

📸 **Screenshot Name**

```
agents-active.png
```

📌 Capture:

Agents page showing active endpoints.

---

# Week 2 – File Integrity Monitoring (FIM)

## Goal

Detect unauthorized modifications to critical system files.

---

# Step 1 – Configure FIM

Edit configuration:

```bash
sudo nano /var/ossec/etc/ossec.conf
```

Add directories:

```
/etc
/bin
/usr/bin
```

Restart manager:

```bash
sudo systemctl restart wazuh-manager
```

---

# Step 2 – Simulate File Modification

Modify system file:

```bash
sudo nano /etc/passwd
```

Save file.

---

# Step 3 – Verify Alert

Open dashboard:

```
Security Events → Alerts
```

Expected alert:

```
File Integrity Monitoring Alert
File changed: /etc/passwd
```

---

## Screenshot 7

📸 **Screenshot Name**

```
fim-alert.png
```

📌 Capture:

Alert showing `/etc/passwd modified`.

---

# Week 3 – Active Response (Blocking Attackers)

## Goal

Automatically block attackers performing brute force attacks.

---

# Step 1 – Configure Active Response

Edit configuration:

```bash
sudo nano /var/ossec/etc/ossec.conf
```

Add:

```
<active-response>
<command>firewall-drop</command>
<location>local</location>
<rules_id>5710</rules_id>
</active-response>
```

Restart Wazuh:

```bash
sudo systemctl restart wazuh-manager
```

---

# Step 2 – Simulate Brute Force Attack

Use **Hydra**.

Example:

```bash
hydra -l root -P rockyou.txt ssh://192.168.1.30
```

This attempts multiple SSH logins.

---

# Step 3 – Detect Attack

Wazuh generates alert:

```
Multiple SSH authentication failures
```

---

## Screenshot 8

📸 **Screenshot Name**

```
ssh-bruteforce-alert.png
```

📌 Capture:

Alert showing SSH brute force attack.

---

# Step 4 – Verify IP Block

Check firewall:

```bash
sudo iptables -L
```

Blocked attacker IP appears.

---

## Screenshot 9

📸 **Screenshot Name**

```
ip-blocked.png
```

📌 Capture:

iptables rule blocking attacker IP.

---

# Week 4 – Threat Simulation using Atomic Red Team

## Goal

Simulate real-world cyber attacks and verify detection.

---

# Step 1 – Install Atomic Red Team

Run in PowerShell:

```powershell
Install-Module -Name Invoke-AtomicRedTeam
```

Import module:

```powershell
Import-Module Invoke-AtomicRedTeam
```

---

# Step 2 – List Available Techniques

```powershell
Get-AtomicTechnique
```

Shows multiple MITRE ATT&CK simulations.

---

## Screenshot 10

📸 **Screenshot Name**

```
atomic-techniques.png
```

📌 Capture:

Output of `Get-AtomicTechnique`.

---

# Step 3 – Run Attack Simulation

Example:

```
T1490 – Inhibit System Recovery
```

Run test:

```powershell
Invoke-AtomicTest T1490
```

This deletes shadow copies.

---

# Step 4 – Detect Attack

Open dashboard.

Alert appears:

```
MITRE ATT&CK Technique T1490 detected
```

---

## Screenshot 11

📸 **Screenshot Name**

```
mitre-attack-alert.png
```

📌 Capture:

Alert showing MITRE ATT&CK technique.

---

# Step 5 – Visualize Threat

Open:

```
Threat Hunting → MITRE ATT&CK
```

Shows attack mapping.

---

## Screenshot 12

📸 **Screenshot Name**

```
mitre-dashboard.png
```

📌 Capture:

MITRE ATT&CK visualization.

---

# Final Results

The SOC monitoring system successfully implemented:

* Endpoint monitoring
* File Integrity Monitoring
* Brute force detection
* Automated IP blocking
* MITRE ATT&CK threat mapping
* Real-world attack simulation

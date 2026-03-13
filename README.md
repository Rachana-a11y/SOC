# Project 2: Enterprise EDR & Threat Hunting Grid (Sentient Shield)

**Objective:**  
Build an enterprise-grade SOC framework for real-time detection and automated response to brute-force and ransomware attacks. Includes File Integrity Monitoring (FIM), MITRE ATT&CK mapping, and active response automation.

**Use Case:**  
Infotact servers are under constant brute-force and ransomware threats. This project demonstrates detection, automated response, and executive-level reporting with MITRE ATT&CK correlation.

---

## Week 1 – Infrastructure & Agent Deployment

**Objective:**  
Deploy Wazuh Manager and agents on Linux and Windows servers, install Sysmon, and validate heartbeat/log reporting.

### Steps & Commands

**1. Wazuh Manager on Linux**
```bash
sudo apt update
sudo apt install wazuh-manager -y
sudo systemctl enable --now wazuh-manager
sudo systemctl status wazuh-manager
```

**2. Linux Agent Deployment**
```bash
sudo apt install wazuh-agent -y
sudo nano /var/ossec/etc/ossec.conf   # configure Wazuh Manager IP
sudo systemctl enable --now wazuh-agent
sudo systemctl status wazuh-agent
```

**3. Windows Agent & Sysmon**

Install Wazuh Agent MSI and configure Manager IP

Install Sysmon:
```bash
Sysmon64.exe -accepteula -i sysmonconfig.xml
```
Explanation:

Agents collect real-time system and log data.

Sysmon enhances Windows telemetry with process, network, and file events.

Continuous heartbeat monitoring ensures agent health.

Gate Check: Verify all agents show Active in Wazuh dashboard.

Screenshots to Include:

week1_agents.png – Wazuh agents Active status

week1_sysmon.png – Sysmon logs showing process/network events

## Week 2 – Detection Rules (FIM & Custom Logic)

Objective:
Configure File Integrity Monitoring (FIM) and custom XML rules for proprietary logs. Enable Vulnerability Detector.

Steps & Commands

1. FIM Configuration
```bash
<syscheck>
  <directories check_all="yes">/etc,/var/www/html,/opt/app/config</directories>
  <frequency>600</frequency>  <!-- check every 10 minutes for enterprise environment -->
</syscheck>
```
2. Custom XML Rule
```bash
<group name="custom-app-logs">
  <rule id="100001" level="10">
    <decoded_as>custom-app</decoded_as>
    <description>Critical configuration change detected</description>
    <frequency>1</frequency>
  </rule>
</group>
```
3. Enable Vulnerability Detector

sudo nano /var/ossec/etc/ossec.conf
sudo systemctl restart wazuh-agent

4. Test Detection

echo "Test modification" >> /var/www/html/index.html

Explanation:

FIM monitors critical files for unauthorized changes.

Custom rules generate high-severity alerts for sensitive application logs.

Vulnerability Detector scans for known CVEs automatically.

Gate Check: High-severity alert should appear within seconds in the Wazuh dashboard.

MITRE ATT&CK Mapping Example:

File modification → T1070 (Indicator Removal)

Unauthorized access → T1078 (Valid Accounts)

Screenshots to Include:

week2_fim_alert.png – FIM alert

week2_custom_rule.png – Custom XML rule triggered

week2_vuln_detector.png – Vulnerability Detector scan results

Week 3 – Active Response (IPS)

Objective:
Automatically block attacker IPs during SSH brute-force attacks.

Steps & Commands

1. Configure Active Response
```xml
<active-response>
  <command>firewalld-drop</command>
  <location>all</location>
  <rules_id>100002,100003</rules_id>
</active-response>
```
2. Test Brute-force Attack

hydra -l root -P passwords.txt ssh://target-server-ip

3. Verify Firewall Blocking
```
sudo firewall-cmd --list-all
sudo tail -f /var/log/messages | grep "Blocked IP"
```
Explanation:

Wazuh detects repeated failed login attempts and triggers mitigation.

Active Response blocks attackers before damage occurs.

MITRE ATT&CK mapping: T1110 – Brute Force.

Gate Check: Attacker IP is automatically blocked; alert appears in dashboard.

Extra Enhancements:

Configure Slack/email notifications for high-severity responses.

Maintain a log of blocked IPs for SOC review.

Screenshots to Include:

week3_bruteforce_alert.png – Brute-force alert

week3_firewall_block.png – Firewall blocked IP logs

Week 4 – Threat Simulation & Kill Chain Visualization

Objective:
Simulate ransomware attacks using Atomic Red Team and visualize alerts in Kibana/OpenSearch.

Steps & Commands

1. Clone Atomic Red Team
``` bash
git clone https://github.com/redcanaryco/atomic-red-team.git
cd atomic-red-team
```
2. Simulate Ransomware (Delete Shadow Volume Copies – T1490)
``` PowerShell
Invoke-AtomicTest T1490 -Target C:\Test
```
3. Map Alerts to MITRE ATT&CK

Shadow Copy deletion → T1490

Process injection → T1055

Credential dumping → T1003

4. Visualize Kill Chain in Kibana/OpenSearch

End-to-end detection: Initial Access → Execution → Detection → Response

Gate Check: Kill Chain visualization confirms detection and response workflow.

Screenshots to Include:

week4_atomic_simulation.png – Atomic Red Team simulation output

week4_kill_chain.png – Kill Chain visualization in Kibana

Wazuh alerts mapped to MITRE ATT&CK

Tools & Technologies

Wazuh Manager & Agents (Linux & Windows)

Sysmon (Windows)

Atomic Red Team

Kibana / OpenSearch

Hydra (Brute-force testing)

MITRE ATT&CK framework

Enterprise Firewall (firewalld / iptables)

Suggested Screenshot Plan
Week	Filename	Description
1	week1_agents.png	Wazuh agents Active status
1	week1_sysmon.png	Sysmon logs showing process/network events
2	week2_fim_alert.png	FIM high-severity alert
2	week2_custom_rule.png	Custom XML rule triggered
2	week2_vuln_detector.png	Vulnerability Detector scan
3	week3_bruteforce_alert.png	Brute-force detection alert
3	week3_firewall_block.png	Firewall blocked IP logs
4	week4_atomic_simulation.png	Atomic Red Team simulation results
4	week4_kill_chain.png	Kill Chain visualization in Kibana/OpenSearch



Project2-SOC-EDR/
├─ README.md
├─ screenshots/
│  ├─ week1_agents.png
│  ├─ week1_sysmon.png
│  ├─ week2_fim_alert.png
│  ├─ week2_custom_rule.png
│  ├─ week2_vuln_detector.png
│  ├─ week3_bruteforce_alert.png
│  ├─ week3_firewall_block.png
│  ├─ week4_atomic_simulation.png
│  └─ week4_kill_chain.png




# Security-OperatiGovernance-Risk-and-Compliance-GRC-mapping-in-a-home-lab-using-Kali-Linux-and-Splunk
This project simulates a Security Operations Center (SOC) with Governance, Risk, and Compliance (GRC) mapping in a home lab using Kali Linux and Splunk. The lab collects, monitors, and analyzes logs for security events and maps alerts to NIST CSF controls.

<img width="975" height="650" alt="image" src="https://github.com/user-attachments/assets/d6b1c85c-76f1-48e0-aadc-74bc33319cc7" />

Folder Structure
text
SOC-GRC-Home-Lab/
│
├── README.md
├── install_splunk.sh
├── configure_forwarder.sh
├── alerts/
│   ├── alerts_configuration.csv
│   └── nist_mapping.json
├── docs/
│   ├── risk_assessment.csv
│   └── lab_schematic.md
├── screenshots/
│   ├── dashboard.png
│   └── alerts.png
└── splunk_configs/
    ├── inputs.conf
    └── outputs.conf
1. README.md
markdown
# SOC/GRC Home Lab Project

A Security Operations Center (SOC) with Governance, Risk, and Compliance (GRC) mapping in a home lab using Kali Linux and Splunk.

## Project Overview
This lab collects, monitors, and analyzes logs for security events and maps alerts to NIST CSF controls.

### Key Features:
- Log collection from Kali Linux (system, authentication, services)
- Alert configuration for suspicious activities
- NIST CSF/ISO 27001 compliance mapping
- SOC dashboards and risk documentation

## Architecture
text
         ┌─────────────┐
         │   Kali VM   │
         │ (Logs Source)│
         └─────┬──────┘
               │ syslogs, auth logs, CRON logs
               ▼
         ┌─────────────┐
         │ Splunk VM   │
         │ (Indexer)   │
         └─────┬──────┘
               │ TCP 9997
               ▼
     ┌────────────────────┐
     │ SOC Dashboard &     │
     │ GRC Documentation   │
     └────────────────────┘
text

## Installation
1. Splunk Enterprise setup:
```bash
./install_splunk.sh
Configure Universal Forwarder:

bash
./configure_forwarder.sh <SPLUNK_INDEXER_IP>
Documentation
Lab Schematic

Risk Assessment

Alert Configuration

text

### 2. install_splunk.sh
```bash
#!/bin/bash
# Splunk Enterprise installation script

VERSION="9.1.2"
SPLUNK_TAR="splunk-${VERSION}-xxx-linux-2.6-x86_64.tgz"
SPLUNK_ADMIN_PASSWORD="ChangeMe123!"

echo "[*] Installing Splunk Enterprise..."
wget -O /tmp/${SPLUNK_TAR} "https://download.splunk.com/products/splunk/releases/${VERSION}/linux/${SPLUNK_TAR}"
sudo tar -xvzf /tmp/${SPLUNK_TAR} -C /opt
sudo /opt/splunk/bin/splunk start --accept-license --answer-yes --no-prompt --seed-passwd ${SPLUNK_ADMIN_PASSWORD}
sudo /opt/splunk/bin/splunk enable boot-start -user splunk
sudo /opt/splunk/bin/splunk enable listen 9997 -auth admin:${SPLUNK_ADMIN_PASSWORD}

echo "[+] Splunk Enterprise installed successfully!"
echo "[*] Access Splunk at https://localhost:8000"
3. configure_forwarder.sh
bash
#!/bin/bash
# Splunk Universal Forwarder configuration script

if [ -z "$1" ]; then
    echo "Usage: $0 <SPLUNK_INDEXER_IP>"
    exit 1
fi

INDEXER_IP=$1
FORWARDER_VERSION="9.1.2"
FORWARDER_DEB="splunkforwarder-${FORWARDER_VERSION}-xxx-linux-2.6-amd64.deb"
SPLUNK_ADMIN_PASSWORD="ChangeMe123!"

echo "[*] Installing Splunk Universal Forwarder..."
wget -O /tmp/${FORWARDER_DEB} "https://download.splunk.com/products/universalforwarder/releases/${FORWARDER_VERSION}/linux/${FORWARDER_DEB}"
sudo dpkg -i /tmp/${FORWARDER_DEB}

echo "[*] Configuring forwarder..."
sudo /opt/splunkforwarder/bin/splunk start --accept-license --answer-yes --no-prompt --seed-passwd ${SPLUNK_ADMIN_PASSWORD}
sudo /opt/splunkforwarder/bin/splunk enable boot-start -user splunk
sudo /opt/splunkforwarder/bin/splunk add forward-server ${INDEXER_IP}:9997 -auth admin:${SPLUNK_ADMIN_PASSWORD}

echo "[*] Adding log monitors..."
sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/auth.log -sourcetype linux_secure -auth admin:${SPLUNK_ADMIN_PASSWORD}
sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/syslog -sourcetype syslog -auth admin:${SPLUNK_ADMIN_PASSWORD}
sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/cron.log -sourcetype linux_cron -auth admin:${SPLUNK_ADMIN_PASSWORD}

sudo /opt/splunkforwarder/bin/splunk restart

echo "[+] Splunk Universal Forwarder configured successfully!"
4. alerts/alerts_configuration.csv
csv
Alert Name,Log Source,Condition,Action,NIST CSF Control
Root Login via CRON,/var/log/auth.log,"Cron session opened for root",Send email / dashboard alert,AU-2
Excessive Failed Logins,/var/log/auth.log,">5 failed login attempts within 5 minutes",Send alert,AC-7
CRON Spike Detection,/var/log/cron.log,">10 cron jobs triggered in 10 minutes",Send alert,CM-6
SSH Brute Force Attempt,/var/log/auth.log,">3 failed SSH attempts from single IP",Send alert,AC-7
Unauthorized Sudo Attempt,/var/log/auth.log,"sudo: user NOT in sudoers",Send alert,AC-2
5. docs/risk_assessment.csv
csv
Risk Scenario,Likelihood,Impact,Risk Level,Mitigation/Alert
Unauthorized root access via CRON,Medium,High,High,Root Login Alert
Brute force login attempts,High,Medium,High,Failed SSH Alert
Excessive cron activity,Medium,Medium,Medium,Cron Spike Alert
Unauthorized privilege escalation,Low,High,High,Sudo Attempt Alert
Malicious scheduled tasks,Medium,High,High,Cron Job Monitoring
6. splunk_configs/inputs.conf
ini
[monitor:///var/log/auth.log]
sourcetype = linux_secure
index = main

[monitor:///var/log/syslog]
sourcetype = syslog
index = main

[monitor:///var/log/cron.log]
sourcetype = linux_cron
index = main
7. splunk_configs/outputs.conf
ini
[tcpout]
defaultGroup = default-autolb-group

[tcpout:default-autolb-group]
server = <INDEXER_IP>:9997

[tcpout-server://<INDEXER_IP>:9997]
8. alerts/nist_mapping.json
json
{
  "alerts": [
    {
      "name": "Root Login via CRON",
      "control": "AU-2",
      "description": "Audit and Accountability - Event Logging",
      "framework": "NIST CSF"
    },
    {
      "name": "Excessive Failed Logins",
      "control": "AC-7",
      "description": "Access Control - Unsuccessful Login Attempts",
      "framework": "NIST CSF"
    },
    {
      "name": "CRON Spike Detection",
      "control": "CM-6",
      "description": "Configuration Management - Configuration Settings",
      "framework": "NIST CSF"
    }
  ]
}
How to Use This Repository:
Clone the repository

Make the scripts executable:

bash
chmod +x *.sh
Update the configuration files with your specific IPs and passwords

Run the installation scripts

Add your dashboard screenshots to the screenshots folder

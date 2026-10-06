# Wazuh SIEM Project (`cysloop/Wazuh-SIEM-Project`)

This repository contains custom configurations, detection rules, and decoders for **Wazuh SIEM**, designed to enhance security monitoring, log analysis, and threat detection.

---

## Project Structure

```text
Wazuh Services Control
To manage, start, or restart the core Wazuh components and dashboard services, use the following commands:

Start / Restart Wazuh Manager:
sudo systemctl start wazuh-manager
(Or using service control):
/var/ossec/bin/wazuh-control restar
Start / Manage Wazuh Dashboard:
sudo systemctl start wazuh-dashboard
Start / Manage Wazuh Indexer (Elasticsearch/OpenSearch core):
sudo systemctl start wazuh-indexer

Wazuh-SIEM-Project/
├── config/              # Core configuration and system settings
│   └── ossec.conf.example
├── decoders/            # Custom log decoders
│   └── local_decoder.xml
├── rules/               # Custom threat detection rules
│   └── local_rules.xml
└── README.md            # Project documentation
Components & Features
Custom Rules (rules/local_rules.xml):

Tailored security rules designed to detect specific malicious behaviors, unauthorized access attempts, and anomalies.

Custom Decoders (decoders/local_decoder.xml):

Specialized log parsers to correctly extract fields from custom or non-standard application logs before triggering alerts.

Configuration Template (config/ossec.conf.example):

Optimized system configuration parameters for agent-server communication and log collection pathways.

Installation & Deployment
To apply these configurations and rules to your Wazuh Manager instance, place the files into their respective directories:

Rules Directory:
/var/ossec/etc/rules/
Decoders Directory:
/var/ossec/etc/decoders/
After copying the files, restart the Wazuh manager service to apply the changes:

sudo systemctl restart wazuh-manager
Author & Context
Developed as part of Security Operations Center (SOC) engineering, threat analysis, and infrastructure hardening workflows.

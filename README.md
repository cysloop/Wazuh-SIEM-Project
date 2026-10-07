# 🛡️ Wazuh SIEM Project  — Detection Engineering & Automated Response

A hands-on **SOC / Blue Team** lab built on **Wazuh**, featuring custom detection rules, custom decoders, file integrity monitoring (FIM), VirusTotal threat-intelligence enrichment, and **Active Response** that isolates a host **automatically or manually** from the Wazuh dashboard.

![Wazuh](https://img.shields.io/badge/SIEM-Wazuh-blue)
![Focus](https://img.shields.io/badge/Focus-SOC%20%7C%20Detection%20Engineering-red)
![MITRE](https://img.shields.io/badge/MITRE%20ATT%26CK-mapped-orange)
![Status](https://img.shields.io/badge/Status-Lab%20Project-green)

---

## 📌 Overview

This project simulates a small enterprise environment to practice the complete SOC workflow:

**Collect → Detect → Alert → Investigate → Respond**

An attacker machine performs reconnaissance against a Windows Server. Wazuh detects it with custom rules, raises a high-severity alert, and an analyst (or an automatic rule) can contain the affected host through Active Response.

---

##  Lab Architecture

| Role | OS | Description |
|------|----|-------------|
| Wazuh Server | Linux | Manager + Indexer + Dashboard |
| Windows Server | Windows Server | Monitored endpoint (Wazuh agent) — main target |
| Ubuntu Server | Ubuntu | Monitored endpoint (Wazuh agent) — also used as the attacker machine for tests |

```
 ┌──────────────────┐   nmap scan    ┌──────────────────┐
 │  Ubuntu Server   │ ─────────────▶ │  Windows Server  │
 │  (Wazuh agent)   │                │  (Wazuh agent)   │
 └────────┬─────────┘                └────────┬─────────┘
          │  logs / events                    │  logs / events / FIM
          └──────────────┐     ┌──────────────┘
                         ▼     ▼
                  ┌──────────────────┐
                  │   Wazuh Server   │
                  │ Manager • Indexer│
                  │    • Dashboard   │
                  └────────┬─────────┘
                           │ Active Response (automatic / manual)
                           ▼
                 Isolate the affected server
```

>  All IP addresses, hostnames and usernames in this repository are sanitized. Replace placeholders such as `<WAZUH_SERVER_IP>` with your own values.

---

##  Features

- **Custom detection rules** — including a **port-scan detection** rule (ID `100101`, **Level 15**) triggered when the Windows Server is scanned with Nmap.
- **Custom decoders** to parse non-standard log sources.
- **File Integrity Monitoring (FIM / Syscheck)** to detect unauthorized file changes.
- **Ransomware detection** — FIM-based detection of abnormal mass file modification/encryption, with **automatic host isolation**.
- **RDP / brute-force login detection** — detects repeated failed logins and password-guessing attempts.
- **Sysmon integration** — deep Windows telemetry (process creation, network connections) forwarded to Wazuh.
- **VirusTotal integration** to enrich file-change alerts with threat intelligence.
- **Host isolation via Active Response** — `isolate-host` / `unisolate-host` scripts:
  - **Automatic**: triggered by a detection rule.
  - **Manual**: launched by the analyst from Wazuh (Dashboard / Dev Tools).
  - Works on **both** the Windows and Ubuntu servers.
- **Active Response whitelist** to prevent isolating trusted hosts.

---

##  Detection Use Cases & MITRE ATT&CK Mapping

| Use Case | Detection Method | Severity | MITRE ATT&CK |
|----------|------------------|----------|--------------|
| Port scan against Windows Server | Custom rule `100101` | Level 15 | T1046 – Network Service Discovery |
| Ransomware-like mass file changes / encryption | FIM (Syscheck) + custom rules + auto isolation | High | T1486 – Data Encrypted for Impact |
| RDP / brute-force login attempts | Windows logon events + custom rules | High | T1110 – Brute Force |
| Suspicious process & network activity | Sysmon events + Wazuh rules | Medium–High | Multiple (Execution, Discovery, C2) |
| Malicious file dropped on endpoint | FIM + VirusTotal | High | T1204 – User Execution |
| Containment of a compromised host | Active Response (`isolate-host`) | — | Response / Containment |

---

##  Repository Structure

```
Wazuh-SIEM-Project/
├── wazuh-project-public/
│   ├── rules/
│   │   └── local_rules.xml          # Custom detection rules
│   ├── decoders/
│   │   └── local_decoder.xml        # Custom log decoders
│   ├── config/
│   │   └── ossec.conf.example       # Sanitized manager config (FIM, VirusTotal, Active Response)
│   └── active-response/             # Host isolation scripts
│       ├── isolate-host.cmd         # Windows – isolate
│       └── unisolate-host.cmd       # Windows – restore
└── README.md
```

> Adjust the tree above to match the exact file names in your repository (e.g. the Ubuntu isolation script).

---

##  Installation & Deployment

### Prerequisites
- A working Wazuh deployment (Manager, Indexer, Dashboard)
- At least one enrolled agent (Windows and/or Linux)
- Root / sudo access on the Wazuh server

### 1. Deploy rules and decoders

```bash
sudo cp rules/local_rules.xml      /var/ossec/etc/rules/local_rules.xml
sudo cp decoders/local_decoder.xml /var/ossec/etc/decoders/local_decoder.xml
```

### 2. Apply the configuration

Review `config/ossec.conf.example`, then merge the relevant blocks (`<syscheck>`, `<integration>`, `<command>`, `<active-response>`) into `/var/ossec/etc/ossec.conf`.
**Do not overwrite your existing configuration blindly.**

For VirusTotal, add your own API key in the `<integration>` block.

### 3. Deploy the isolation scripts to the agents

- **Windows agent:** copy `isolate-host.cmd` and `unisolate-host.cmd` to  
  `C:\Program Files (x86)\ossec-agent\active-response\bin\`
- **Linux agent:** copy the isolation script to `/var/ossec/active-response/bin/` and make it executable:

```bash
sudo chmod 750 /var/ossec/active-response/bin/<isolation-script>
sudo chown root:wazuh /var/ossec/active-response/bin/<isolation-script>
```

Restart the agents afterwards.

### 4. Restart the manager

```bash
sudo systemctl restart wazuh-manager
# or
sudo /var/ossec/bin/wazuh-control restart
```

---

##  Testing

### Port-scan detection
From the attacker machine:

```bash
nmap -sS -p- <TARGET_WINDOWS_IP>
```

**Expected result:** a **Level 15** alert from rule `100101` in the Wazuh Dashboard.

### Manual isolation
Run the `isolate-host` active response against an agent ID from **Dev Tools** in the dashboard, and use `unisolate-host` to restore connectivity.

### Useful commands

```bash
sudo tail -f /var/ossec/logs/alerts/alerts.json     # live alerts
sudo tail -f /var/ossec/logs/active-responses.log   # active response activity
sudo /var/ossec/bin/wazuh-logtest                   # test rules against sample logs
```

---

##  Wazuh Services Cheat Sheet

```bash
sudo systemctl start|restart|status wazuh-manager
sudo systemctl start|restart|status wazuh-indexer
sudo systemctl start|restart|status wazuh-dashboard
sudo /var/ossec/bin/wazuh-control restart
```

---

##  Screenshots

_Add screenshots to `docs/screenshots/` and link them here:_

| Scenario | Preview |
|----------|---------|
| Port-scan alert (Level 15) | `![port scan](docs/screenshots/port-scan-alert.png)` |
| FIM alert | `![fim](docs/screenshots/fim-alert.png)` |
| Host isolation | `![isolation](docs/screenshots/isolation.png)` |

---

##  Roadmap

- [x] Wazuh server with Windows & Linux agents
- [x] Custom port-scan detection rule (Level 15)
- [x] VirusTotal integration
- [x] Host isolation — automatic and manual
- [x] FIM-based ransomware detection with automatic isolation on abnormal activity
- [x] Brute-force / RDP login detection
- [x] Sysmon integration for deeper Windows telemetry
- [ ] SOC triage dashboards
- [ ] Additional MITRE ATT&CK-mapped detections

---

##  Skills Demonstrated

`SIEM` · `Detection Engineering` · `Wazuh Rules & Decoders (XML)` · `File Integrity Monitoring` · `Active Response` · `Sysmon` · `Ransomware & Brute-Force Detection` · `Threat Intelligence Enrichment` · `MITRE ATT&CK Mapping` · `Windows & Linux Log Analysis` · `Incident Response`

---

##  Disclaimer

This project is for **educational and lab purposes only**. Run attack simulations only on systems you own or are explicitly authorized to test.

---

## 👤 Author

**cysloop** — Cybersecurity / SOC   
🔗 GitHub: [@cysloop](https://github.com/cysloop)

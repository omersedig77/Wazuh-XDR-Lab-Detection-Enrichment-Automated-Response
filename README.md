# 🛡️ Wazuh XDR Lab: Detection, Enrichment & Automated Response

> A hands-on Security Operations Center (SOC) lab built on **Wazuh XDR**, focused on the modern analyst workflow: **detect → enrich → respond — automatically**.

![Status](https://img.shields.io/badge/status-active-success)
![Wazuh](https://img.shields.io/badge/Wazuh-4.9-blue)
![Platform](https://img.shields.io/badge/platform-Ubuntu%2022.04-orange)
![License](https://img.shields.io/badge/license-MIT-green)
![SOAR](https://img.shields.io/badge/SOAR-n8n-purple)

---

## 📖 Overview

This lab simulates a small enterprise SOC environment where telemetry from Windows and Linux endpoints is centralized in **Wazuh XDR**. Unlike a traditional SIEM setup, this project goes further: it **automatically responds** to threats using Active Response, **enriches** indicators with multiple threat intelligence sources, and produces **professional L1 triage reports** for every verified incident.

The lab is built entirely with **free and open-source tools**.

> 🔗 **Companion Project:** [SOC Homelab with Splunk Enterprise](#) *(Project 1)* — focuses on SIEM deployment, custom detection engineering, and dashboards. This second lab extends those concepts into **response automation, threat intelligence, and SOAR**.

---

## 🏗️ Lab Architecture Diagram

<img width="1408" height="768" alt="01-lab-network-diagram" src="https://github.com/user-attachments/assets/d90f4ad6-d5dd-48e6-872a-2e057d99d042" />

---

## 🎯 Project Objectives

- ✅ Deploy a functional Wazuh XDR stack (Manager + Indexer + Dashboard)
- ✅ Onboard Windows and Linux endpoints via Wazuh agents
- ✅ Configure Active Response for automated containment of brute-force attacks
- ✅ Integrate VirusTotal for automated IOC enrichment
- ✅ Enable Vulnerability Detection and prioritize CVEs
- ✅ Build a lightweight SOAR pipeline with n8n and Discord
- ✅ Develop 3 automated response playbooks (SSH, Malware, Phishing)
- ✅ Produce professional L1 triage reports

---

## 🖥️ Lab Architecture

| Component | OS | IP | Role |
|---|---|---|---|
| Wazuh Server | Ubuntu Server 22.04 | 192.168.50.10 | XDR Manager + Indexer + Dashboard |
| Windows Endpoint | Windows 10 Pro | 192.168.50.20 | Monitored endpoint (agent + Sysmon) |
| Linux Endpoint | Ubuntu 22.04 | 192.168.50.30 | Monitored endpoint (agent + auditd) |
| Kali Linux | Kali | 192.168.50.40 | Attack simulation |
| pfSense | pfSense | 192.168.50.1 | Gateway / Firewall |

> 📐 Detailed architecture: [`docs/01-architecture.md`](docs/01-architecture.md)

---

## ⚡ Key Capabilities

| Capability | Status |
|---|---|
| Centralized log collection (Wazuh XDR) | ✅ Done |
| Windows event monitoring | ✅ Done |
| Linux log monitoring | ✅ Done |
| Active Response (auto-block) | ✅ Done |
| VirusTotal IOC enrichment | ✅ Done |
| Vulnerability Detection | ✅ Done |
| SOAR pipeline (n8n + Docker) | ✅ Done |
| SSH Brute Force enrichment playbook | ✅ Done |
| Malware Response playbook | ✅ Done |
| Phishing Triage playbook | ✅ Done |
| L1 triage reports | ✅ Done |

---

## 🚀 SOAR Integration

A lightweight SOAR layer built with **n8n** (running in Docker) automates the SOC L1 triage workflow:

- Ingests alerts from Wazuh via custom Python integration
- Enriches IOCs via **VirusTotal**, **AbuseIPDB**, and **urlscan.io**
- Applies conditional decision logic (severity routing)
- Delivers color-coded alerts to **Discord** in real time

### Playbooks

| Playbook | Trigger | Enrichment | Output |
|---|---|---|---|
| **SSH Brute Force** | Rule 5763 / auth failures | VirusTotal (IP), AbuseIPDB | 3-tier Discord alert (High/Suspicious/Internal) |
| **Malware Response** | Rule 87105 (VT detects file) | VirusTotal (file hash) | 4-tier Discord alert (Confirmed/Likely/Suspicious/Clean) |
| **Phishing Triage** | Custom webhook | VirusTotal (domain), AbuseIPDB (IP), urlscan.io (URL) | 3-tier Discord alert (Malicious/Suspicious/Clean) |

> 📚 See [`docs/09-soar-architecture.md`](docs/09-soar-architecture.md) for architecture details.

---

## 🧪 Verified Attack Scenarios

| Attack | Detection Rule | SOAR Playbook | Result |
|---|---|---|---|
| SSH Brute Force (Hydra from Kali) | 5763 | SSH Brute Force Playbook | Auto-contained via Active Response + Discord alert |
| Malware File Drop (EICAR) | 87105 | Malware Response Playbook | Enriched + Discord alert with ISOLATE ENDPOINT recommendation |
| Phishing Email (Simulated) | Webhook | Phishing Triage Playbook | Multi-source enrichment + Discord triage alert |

---

## 📚 Documentation

| # | Document | Description |
|---|---|---|
| 01 | [Architecture](docs/01-architecture.md) | Network design, VM layout, ports |
| 02 | [Lab Setup](docs/02-lab-setup.md) | VMware config, VM specs, IPs |
| 03 | [Wazuh Installation](docs/03-wazuh-installation.md) | Manager + Indexer + Dashboard |
| 04 | [Agent Deployment](docs/04-agent-deployment.md) | Windows and Linux agents |
| 05 | [Active Response](docs/05-active-response.md) | Automated firewall-block |
| 06 | [VirusTotal Integration](docs/06-virustotal-integration.md) | File IOC enrichment |
| 07 | [Vulnerability Detection](docs/07-vulnerability-detection.md) | CVE scanning and triage |
| 08 | [SOAR Architecture](docs/09-soar-architecture.md) | n8n + Docker + Discord pipeline |
| 09 | [SSH Brute Force Playbook](docs/10-ssh-bruteforce-playbook.md) | VT + AbuseIPDB enrichment |
| 10 | [Malware Response Playbook](docs/11-malware-response-playbook.md) | VT file enrichment + action |
| 11 | [Phishing Triage Playbook](docs/12-phishing-triage-playbook.md) | Multi-source email triage |
| 12 | [L1 Triage Reports](triage-reports/README.md) | Professional alert triage documentation |

---

## 📋 L1 Triage Report Library

The [`triage-reports/`](triage-reports/) folder contains professional L1 triage reports for real alerts generated in the lab.

| # | Report | Verdict | Severity |
|---|---|---|---|
| 001 | [SSH Brute Force](triage-reports/alert-001-ssh-bruteforce.md) | True Positive | High |
| 002 | [Malware File Detected](triage-reports/alert-002-malware-detection.md) | True Positive (Test) | Critical |
| 003 | [Phishing Email Reported](triage-reports/alert-003-phishing-email.md) | Suspicious | Medium |
| 004 | [Active Response Auto-Block](triage-reports/alert-004-active-response-block.md) | True Positive | High |
| 005 | [Critical CVE Detected](triage-reports/alert-005-vulnerability-cve.md) | True Positive | Critical |

---

## 🔧 Tech Stack

- **SIEM/XDR:** Wazuh 4.9
- **Log storage:** OpenSearch (Wazuh Indexer)
- **Visualization:** Wazuh Dashboard
- **SOAR:** n8n (self-hosted, Docker)
- **Container runtime:** Docker 29.8.1 + Docker Compose v5.5.1
- **Threat Intel APIs:** VirusTotal, AbuseIPDB, urlscan.io
- **Notification:** Discord webhooks
- **Endpoints:** Windows 10 Pro, Ubuntu 22.04
- **Attack simulation:** Kali Linux, Hydra
- **Firewall/Gateway:** pfSense
- **Virtualization:** VMware Workstation 17 Player

---

## 🎓 Skills Demonstrated

- SIEM/XDR deployment and operation (Wazuh)
- Windows Event Log and Sysmon analysis
- Linux authentication log analysis
- Detection engineering (custom rules)
- **Automated incident response (Active Response)**
- **Threat intelligence enrichment (VirusTotal, AbuseIPDB, urlscan.io)**
- Vulnerability management (CVE triage)
- **SOAR workflow design (n8n)**
- **Docker container orchestration**
- **Python scripting for security automation**
- **Webhook integration between SIEM and SOAR**
- Multi-source decision logic
- MITRE ATT&CK mapping
- **Professional incident documentation (L1 triage reports)**

---

## 🔐 Security Notes

- All API keys are stored as n8n credentials, not in workflow code
- The lab runs on an isolated internal network (`192.168.50.0/24`)
- Attacks are performed exclusively against lab VMs owned by the author
- No real production data or live systems are involved

---

## 👤 Author

**[Omer Sedig]**
- LinkedIn: [https://www.linkedin.com/in/omersedig/]
- GitHub: [https://github.com/omersedig77]

---

## 📜 License

This project is licensed under the MIT License — see [LICENSE](LICENSE) for details.

> ⚠️ **Disclaimer:** This lab is for **educational purposes only**. All attacks are performed against isolated lab VMs owned by the author.

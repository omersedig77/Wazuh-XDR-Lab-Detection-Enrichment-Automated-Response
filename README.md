# 🛡️ Wazuh XDR Lab: Detection, Enrichment & Automated Response

> A hands-on Security Operations Center (SOC) lab built on **Wazuh XDR**, focused on the modern analyst workflow: **detect → enrich → respond — automatically**.

![Status](https://img.shields.io/badge/status-in%20progress-yellow)
![Wazuh](https://img.shields.io/badge/Wazuh-4.9-blue)
![Platform](https://img.shields.io/badge/platform-Ubuntu%2022.04-orange)
![License](https://img.shields.io/badge/license-MIT-green)

---

## 📖 Overview

This lab simulates a small enterprise SOC environment where telemetry from Windows and Linux endpoints is centralized in **Wazuh XDR**. Unlike a traditional SIEM setup, this project goes one step further: it **automatically responds** to threats using Active Response, **enriches** indicators with VirusTotal, and produces **L1-style triage reports** for every verified incident.

The lab is built entirely with **free and open-source tools**.

> 🔗 **Companion Project:** [SOC Homelab with Splunk Enterprise](#) *(Project 1)* — focuses on SIEM deployment, custom detection engineering, and dashboards. This second lab extends those concepts into **response automation and threat intelligence**.

---

## 🎯 Project Objectives

- ✅ Deploy a functional Wazuh XDR stack (Manager + Indexer + Dashboard)
- ✅ Onboard Windows and Linux endpoints via Wazuh agents
- ✅ Configure **Active Response** for automated containment of brute-force attacks
- ⏳ Integrate **VirusTotal** for automated IOC enrichment
- ⏳ Enable **Vulnerability Detection** and prioritize CVEs
- ⏳ Simulate realistic attacks using **Atomic Red Team**
- ⏳ Produce professional **L1 triage reports**

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
| Centralized log collection (Wazuh XDR) | ✅ |
| Windows event monitoring | ✅ |
| Linux log monitoring | ✅ |
| **Active Response (auto-block)** | ✅ |
| **VirusTotal IOC enrichment** | ✅ |
| **Vulnerability Detection** | ✅ |
| Attack simulation with Atomic Red Team | ⏳ |
| SOAR pipeline (n8n) | ✅ |
| SSH Brute Force enrichment playbook | ✅ |
| Malware Response playbook | ✅ |
| Phishing Triage playbook | ⏳ |
| L1 triage reports | ⏳ |

---

## 📚 Documentation

| # | Document | Description |
|---|---|---|
| 01 | [Architecture](docs/01-architecture.md) | Network design, VM layout, ports |
| 02 | [Lab Setup](docs/02-lab-setup.md) | VMware config, VM specs, IPs |
| 03 | [Wazuh Installation](docs/03-wazuh-installation.md) | Manager + Indexer + Dashboard |
| 04 | [Agent Deployment](docs/04-agent-deployment.md) | Windows & Linux agents |
| 05 | [Active Response](docs/05-active-response.md) | Automated firewall-block |
| 06 | [VirusTotal Integration](docs/06-virustotal-integration.md) | File IOC enrichment |
| 07 | [Vulnerability Detection](docs/07-vulnerability-detection.md) | CVE scanning & triage |
| 08 | Attack Simulations | ⏳ Coming |
| 09 | [SOAR Architecture](https://docs/09-soar-architecture.md) | n8n + Docker + Discord pipeline |
| 10 | [SSH Brute Force Playbook](https://docs/10-ssh-bruteforce-playbook.md) | VT + AbuseIPDB enrichment |
| 11 | [Malware Response Playbook](https://docs/11-malware-response-playbook.md) | VT file enrichment + action |
| 12 | Phishing Triage playbook | ⏳ Coming |
| 13 | L1 triage reports | ⏳ Coming |

---

## 🧪 Verified Attack Scenarios

| Attack | Rule Triggered | Active Response | Date |
|---|---|---|---|
| SSH Brute Force (Hydra from Kali) | 5763 | ✅ firewall-drop | 2026-09-13 |

> 📄 Triage report: [`triage-reports/alert-001-ssh-bruteforce.md`](triage-reports/alert-001-ssh-bruteforce.md)

---

## 🔧 Tech Stack

- **SIEM/XDR:** Wazuh 4.9
- **Log storage:** OpenSearch (Wazuh Indexer)
- **Visualization:** Wazuh Dashboard
- **Endpoints:** Windows 10 Pro, Ubuntu 22.04
- **Attack simulation:** Kali Linux, Hydra, Atomic Red Team
- **Firewall/Gateway:** pfSense
- **Virtualization:** VMware Workstation 17 Player

---

## 🎓 Skills Demonstrated

- SIEM deployment and operation (Wazuh XDR)
- Windows Event Log and Sysmon analysis
- Linux authentication log analysis
- Detection engineering (custom rules)
- **Automated incident response (Active Response)**
- **Threat intelligence enrichment (VirusTotal)**
- Vulnerability management (CVE triage)
- MITRE ATT&CK mapping
- Professional incident documentation (L1 triage reports)

---

## 👤 Author

**Omer Adam**
- LinkedIn: https://www.linkedin.com/in/omersedig/
- GitHub: https://github.com/omersedig77/

---

## 📜 License

MIT — see [LICENSE](LICENSE) for details.

> ⚠️ **Disclaimer:** This lab is for **educational purposes only**. All attacks are performed against isolated lab VMs owned by the author.

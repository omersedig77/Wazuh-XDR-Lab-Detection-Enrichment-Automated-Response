# 01 - Architecture

## Overview

The Wazuh XDR lab is built on VMware Workstation 17 Player. All VMs communicate over an internal host-only network (`VMnet2`, 192.168.50.0/24) behind a pfSense gateway. A second adapter (`VMnet8`, NAT) provides internet access to the Wazuh server for feed updates and VirusTotal API calls.

## Architecture Diagram
<img width="693" height="633" alt="image" src="https://github.com/user-attachments/assets/a91d4a12-97d7-444b-8ce9-20e37d7a5b1f" />

## IP Addressing

| Host | IP | Network | Notes |
|---|---|---|---|
| pfSense LAN | 192.168.50.1 | VMnet2 | Gateway |
| **Wazuh Server** | **192.168.50.10** | VMnet2 | Static LAN |
| Wazuh Server (WAN) | 192.168.159.138 | VMnet8 | DHCP (Internet only) |
| Windows Endpoint | 192.168.50.20 | VMnet2 | Static |
| Linux Endpoint | 192.168.50.30 | VMnet2 | Static |
| Kali | 192.168.50.40 | VMnet2 | Static |

## Ports

| Source | Destination | Port | Protocol | Purpose |
|---|---|---|---|---|
| Windows Agent | Wazuh Manager | 1514 | TCP | Agent events |
| Windows Agent | Wazuh Manager | 1515 | TCP | Agent enrollment |
| Linux Agent | Wazuh Manager | 1514 | TCP | Agent events |
| Linux Agent | Wazuh Manager | 1515 | TCP | Agent enrollment |
| pfSense | Wazuh Manager | 5514 | UDP | Syslog |
| Analyst (host) | Wazuh Dashboard | 443 | TCP | Web UI |

> ⚠️ **Note:** The Wazuh Dashboard is accessed from the host via the VMnet8 IP (`https://192.168.159.138`) because the VMware Workstation Player host-only adapter for VMnet2 has connectivity issues that prevent direct access from the host.

## Design Decisions

- **Ubuntu Server (not Desktop)** for Wazuh → lower resource usage, matches enterprise deployment
- **Dual NIC on Wazuh Server** → LAN for agents, WAN for internet (updates, VirusTotal)
- **Sysmon on Windows** → deeper process-level telemetry than default Event Log
- **auditd on Linux** → rich execve and auth telemetry
- **pfSense on VMnet2** → realistic enterprise topology

## Data Flow

1. Endpoints generate security events (auth, process, network, file)
2. Wazuh agents forward events to the Wazuh Manager (`192.168.50.10:1514/tcp`)
3. Manager decodes events and applies rules
4. Alerts are stored in Wazuh Indexer and displayed in the Dashboard
5. For high-severity rules, **Active Response** commands are sent back to the endpoint
6. Active Response executes locally (e.g., `iptables DROP`) and reports the result

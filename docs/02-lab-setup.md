# 02 - Lab Setup

## Virtualization Platform

**VMware Workstation 17 Player** on a Windows host.

## Virtual Networks

| Network | Type | Subnet | Purpose |
|---|---|---|---|
| VMnet2 | Host-only | 192.168.50.0/24 | Internal SOC lab |
| VMnet8 | NAT | 192.168.159.0/24 | Internet connectivity for Wazuh |

## Virtual Machines

### Wazuh Server

| Property | Value |
|---|---|
| OS | Ubuntu Server 22.04.5 LTS |
| RAM | 8 GB |
| CPU | 4 cores |
| Disk | 50 GB |
| NIC 1 | VMnet2 → 192.168.50.10/24 (static) |
| NIC 2 | VMnet8 → DHCP (192.168.159.x) |

### Windows Endpoint

| Property | Value |
|---|---|
| OS | Windows 10 Pro (10.0.19045.6466) |
| RAM | 4 GB |
| NIC | VMnet2 → 192.168.50.20/24 |
| Role | Monitored target |
| Software | Sysmon, Wazuh Agent 4.9.0 |

### Linux Endpoint

| Property | Value |
|---|---|
| OS | Ubuntu Desktop 22.04.5 LTS |
| RAM | 2 GB |
| NIC | VMnet2 → 192.168.50.30/24 |
| Role | Monitored target |
| Software | Wazuh Agent 4.9.2 |

### Kali

| Property | Value |
|---|---|
| OS | Kali Linux |
| RAM | 2 GB |
| NIC | VMnet2 → 192.168.50.40/24 |
| Role | Attacker |

### pfSense

| Property | Value |
|---|---|
| OS | pfSense 2.7.x |
| RAM | 1 GB |
| NIC 1 | VMnet2 (LAN) → 192.168.50.1 |
| NIC 2 | VMnet8 (WAN) → DHCP |

## Networking Notes

- Ubuntu Server 22.04 uses **Netplan** for network configuration. The file `/etc/netplan/00-installer-config.yaml` (or `50-cloud-init.yaml`) contains static configuration for `ens33` (VMnet2) and DHCP for `ens34` (VMnet8).
- The host reaches the Wazuh Dashboard via the VMnet8 IP: `https://192.168.159.138`.

## Host Networking Configuration

- VMnet2 was configured as **host-only** with DHCP **disabled**.
- To allow the host to reach VMnet2, the host's `VMware Network Adapter VMnet2` must be enabled with a static IP of `192.168.50.1`.

## Snapshot Strategy

Snapshots were taken at each milestone:
- `snap-01-ubuntu-installed`
- `snap-02-wazuh-installed`
- `snap-03-agents-onboarded`
- `snap-04-active-response-configured`

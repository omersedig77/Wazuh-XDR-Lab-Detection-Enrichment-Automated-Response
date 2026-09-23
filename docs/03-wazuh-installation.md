# 03 - Wazuh Installation

## Overview

Wazuh was deployed using the official **all-in-one** installer, which provisions **Wazuh Manager**, **Wazuh Indexer** (OpenSearch), and **Wazuh Dashboard** on a single Ubuntu Server 22.04 host.

## Prerequisites

- Ubuntu Server 22.04.5 LTS
- 8 GB RAM, 4 CPU cores, 50 GB disk
- Static IP `192.168.50.10/24` on VMnet2
- Internet access via second NIC (VMnet8)

## Step 1 — Install Dependencies

```bash
sudo apt update
sudo apt install -y curl apt-transport-https unzip wget libcap2-bin \
    software-properties-common lsb-release gnupg2
```

## Step 2 — Download the Wazuh Installer

```bash
cd ~
wget --tries=10 --timeout=30 https://packages.wazuh.com/4.9/wazuh-install.sh
chmod 744 wazuh-install.sh
```

## Step 3 — Run the All-in-One Installer

```bash
sudo bash ./wazuh-install.sh -a
```

The -a flag installs Manager + Indexer + Dashboard on the same host.

⏱️ Duration: ~15–20 minutes.

## Step 4 — Save Admin Credentials
The installer prints the admin password at the end. Save it immediately.

If you missed it:

```bash
sudo tar -O -xvf wazuh-install-files.tar wazuh-install-files/wazuh-passwords.txt
```

## Step 5 — Verify Services

```bash
sudo systemctl status wazuh-manager --no-pager
sudo systemctl status wazuh-indexer --no-pager
sudo systemctl status wazuh-dashboard --no-pager
```
All should show active (running).

## Step 6 — Verify Listening Ports

```bash
sudo ss -tlnp | grep -E '443|1514|1515|55000'
```

Port	Service
443	Dashboard
1514	Agent events
1515	Agent enrollment
55000	Wazuh API

## Step 7 — Configure the Manager for Remote Agents

By default, the manager binds to 127.0.0.1. Edit /var/ossec/etc/ossec.conf and add <local_ip>0.0.0.0</local_ip> inside <remote>:

```bash
<remote>
  <local_ip>0.0.0.0</local_ip>
  <connection>secure</connection>
  <port>1514</port>
  <protocol>tcp</protocol>
</remote>
```

Restart:

```bash
sudo systemctl restart wazuh-manager
```

## Step 8 — Access the Dashboard

From the host browser:

```bash
https://192.168.159.138
```
Login: admin / (saved password)

## Troubleshooting

| Issue |	Fix |
|---|---|
| API timeout on first login |	Increase timeout in /usr/share/wazuh-dashboard/data/wazuh/config/wazuh.yml (timeout: 100000) |
| Dashboard 500 errors |	Restart dashboard; if needed, delete .kibana_* indices |
| Agent can't connect |	Verify <local_ip>0.0.0.0</local_ip> is present |
| Unknown configuration key error |	Remove invalid keys from opensearch_dashboards.yml |



# 04 - Agent Deployment

## Windows 10 Agent

### Download the Installer

Download on the Windows endpoint:

https://packages.wazuh.com/4.x/windows/wazuh-agent-4.9.0-1.msi

### Silent Install

Open **Command Prompt as Administrator**:

```cmd
cd %USERPROFILE%\Downloads
msiexec.exe /i wazuh-agent-4.9.0-1.msi /q WAZUH_MANAGER="192.168.50.10" WAZUH_AGENT_NAME="Windows10-Endpoint"
```

Start the Service

```cmd
net start WazuhSvc
sc query WazuhSvc
```
Expected: STATE : 4 RUNNING.

## Verify in Dashboard
The agent appears in the Agents page with status Active.

### Linux Endpoint (Ubuntu 22.04)
Why not the standard apt repo?
The 4.x apt repository serves the latest agent version. Because the lab manager runs 4.9.0, an agent newer than the manager will be rejected (Agent version must be lower or equal to manager version). Install the matching version directly from a .deb:

#### Install

```bash
cd /tmp
wget https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.9.2-1_amd64.deb

sudo WAZUH_MANAGER='192.168.50.10' WAZUH_AGENT_NAME='Linux-Endpoint' \
    dpkg -i ./wazuh-agent_4.9.2-1_amd64.deb
```
When prompted about the service file, answer Y to install the maintainer's version.

#### Fix the Manager IP

The .deb install does not substitute the WAZUH_MANAGER variable. Edit:

```bash
sudo nano /var/ossec/etc/ossec.conf
```

Find <address>MANAGER_IP</address> and replace with:

```xml
<address>192.168.50.10</address>
```

#### Start the Agent

```bash
sudo systemctl daemon-reload
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
sudo systemctl status wazuh-agent --no-pager
```

#### Verify Connection

```bash
sudo tail -25 /var/ossec/logs/ossec.log
```

Look for:
```
INFO: (4102): Connected to the server ([192.168.50.10]:1514/tcp)
```

#### Verify on the Wazuh Server

```bash
sudo /var/ossec/bin/agent_control -l
```

Expected:
```
   ID: 000, Name: wazuh-server (server), IP: 127.0.0.1, Active/Local
   ID: 001, Name: Windows10-Endpoint, IP: 192.168.50.20, Active
   ID: 003, Name: Linux-Endpoint, IP: 192.168.50.30, Active
```

### Troubleshooting

Issue	Fix
Agent version newer than manager	Install matching .deb (see above)
MANAGER_IP placeholder in config	Manually edit ossec.conf
Agent stuck on "Pending"	Restart agent; verify ping 192.168.50.10
Dashboard doesn't show agent	Click Refresh on the Agents page






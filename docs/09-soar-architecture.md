# 09 - SOAR Architecture

## Overview

This document describes the SOAR (Security Orchestration, Automation, and Response) layer built on top of the Wazuh XDR deployment. It automates the SOC L1 triage workflow: alerts from Wazuh are enriched with external threat intelligence and delivered to Discord in real time.

The design is fully open-source, containerized, and runs on the same Ubuntu Server VM hosting Wazuh.

## Architecture Diagram

    ┌─────────────────────┐
    │    Wazuh XDR        │
    │   (Alert Source)    │
    └──────────┬──────────┘
               │
               │ Integration script POSTs alert JSON
               ▼
    ┌─────────────────────┐
    │   n8n (SOAR)        │
    │   Webhook Receiver  │
    └──────────┬──────────┘
               │
               │ Enrichment + Decision Logic
               ▼
    ┌─────────────────────┐
    │  External APIs      │
    │  - VirusTotal       │
    │  - AbuseIPDB        │
    └──────────┬──────────┘
               │
               │ Formatted alert
               ▼
    ┌─────────────────────┐
    │   Discord           │
    │  (Notification)     │
    └─────────────────────┘

## Components

| Component | Purpose | Version | Deployment |
|---|---|---|---|
| Docker Engine | Container runtime | 29.8.1 | Ubuntu Server 22.04 |
| Docker Compose | Multi-container orchestration | v5.5.1 | Ubuntu Server 22.04 |
| n8n | SOAR workflow automation | Latest (2.39.7) | Docker container |
| Wazuh Manager | SIEM alert source | 4.9.0 | Ubuntu Server 22.04 |
| VirusTotal API | File and IP reputation | v3 API | External (free tier) |
| AbuseIPDB API | IP reputation | v2 API | External (free tier) |
| Discord Webhooks | Alert delivery | N/A | External (free tier) |

## Host Placement

All SOAR components run on the **Wazuh Server VM** at `192.168.50.10`. This keeps the lab on a single host and reduces resource consumption.

| Service | Container | Port | URL |
|---|---|---|---|
| n8n | n8n | 5678 | http://192.168.159.138:5678 |

## Directory Layout

    /opt/soar/
    └── n8n/
        └── docker-compose.yml

## n8n Deployment

The n8n container is deployed via Docker Compose with a named volume for persistence.

### docker-compose.yml

```yaml
version: '3.8'

services:
  n8n:
    image: n8nio/n8n:latest
    container_name: n8n
    restart: unless-stopped
    ports:
      - "5678:5678"
    environment:
      - N8N_HOST=192.168.159.138
      - N8N_PORT=5678
      - N8N_PROTOCOL=http
      - WEBHOOK_URL=http://192.168.159.138:5678
      - GENERIC_TIMEZONE=UTC
      - N8N_SECURE_COOKIE=false
      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER=admin
      - N8N_BASIC_AUTH_PASSWORD=soar-lab-password
      - N8N_RUNNERS_ENABLED=true
      - N8N_BLOCK_ENV_ACCESS_IN_NODE=false
    volumes:
      - n8n-data:/home/node/.n8n
volumes:
  n8n-data:
```

### Start the Service
```bash
cd /opt/soar/n8n
sudo docker compose up -d
```

### Verify
```bash
sudo docker ps
sudo docker logs n8n --tail 30
```

Access the UI at ```http://192.168.159.138:5678``` with credentials ```admin / soar-lab-password```.

## Wazuh → n8n Integration
Wazuh uses a custom Python integration script to forward specific alerts to n8n webhooks.

### Integration Script
Located at ```/var/ossec/integrations/custom-n8n.py``` (SSH brute-force playbook) and ```/var/ossec/integrations/custom-n8n-malware.py``` (malware playbook).

Both scripts POST alert JSON to the respective n8n webhook URL.

### Script Permissions
```bash
sudo chmod 750 /var/ossec/integrations/custom-n8n.py
sudo chown root:wazuh /var/ossec/integrations/custom-n8n.py
sudo chmod 750 /var/ossec/integrations/custom-n8n-malware.py
sudo chown root:wazuh /var/ossec/integrations/custom-n8n-malware.py
```

### Integration Registration in ossec.conf

- SSH brute force:

```xml
<ossec_config>
  <integration>
    <name>custom-n8n.py</name>
    <hook_url>http://192.168.159.138:5678/webhook/wazuh-alerts</hook_url>
    <level>7</level>
    <alert_format>json</alert_format>
  </integration>
</ossec_config>
```

- Malware:

```xml
<ossec_config>
  <integration>
    <name>custom-n8n-malware.py</name>
    <hook_url>http://192.168.159.138:5678/webhook/wazuh-malware</hook_url>
    <rule_id>87105</rule_id>
    <alert_format>json</alert_format>
  </integration>
</ossec_config>
```
- Restart the manager after any change:

``` bash
sudo systemctl restart wazuh-manager
```

## Discord Setup
A Discord server named SOC Alerts is used for notifications. A channel webhook named Wazuh SOAR Alerts receives messages from n8n.

The webhook URL follows the format:

```text
https://discord.com/api/webhooks/<id>/<token>
```

n8n sends alerts as rich embeds with color-coded severity and structured fields.

## Alert Flow

1. An attack occurs on a monitored endpoint
2. Wazuh agent forwards events to the Wazuh Manager
3. Wazuh decodes the events and applies detection rules
4. Rules with matching integration criteria trigger the custom script
5. The script POSTs the alert JSON to n8n's webhook
6. n8n parses the alert, enriches it via external APIs, applies decision logic
7. n8n sends a formatted Discord embed with the enriched alert
8. A SOC analyst reviews the Discord notification and acts

## Design Decisions

- n8n over TheHive/Cortex: n8n alone fits the RAM budget (8 GB Wazuh VM), avoids heavyweight Java stacks. TheHive was deferred to future work.

- Discord over Email/Slack: Discord webhooks are free, fast, and support rich embeds without OAuth setup.

- HTTP Request node over native Discord node: The n8n Discord node in v2.39.7 had a bug (sendLegacy:undefined). Using a raw HTTP Request node bypasses the bug and provides identical output.

- Custom Python integration script: Wazuh's built-in integrations do not support arbitrary webhooks. A custom script gives full control over the payload and target URL.

- Separate webhook paths per playbook: Each playbook uses a distinct path (/wazuh-alerts, /wazuh-malware) so alerts route to the correct workflow.

## URLs

Service	URL	Credentials
n8n UI	http://192.168.159.138:5678	admin / soar-lab-password
Discord	https://discord.com	Personal account

## Skills Demonstrated

- Docker container orchestration

- Workflow automation with n8n

- Webhook-based integration between SIEM and SOAR

- Python scripting for security automation

- Multi-source threat intelligence enrichment

- Conditional routing and decision logic

- Rich notification formatting

- Full-stack SOC pipeline design

## Limitations and Future Work

- No case management: TheHive not deployed due to RAM constraints. Cases are tracked in Discord and documentation.

- Free-tier API limits: VirusTotal allows 4 requests/minute; AbuseIPDB allows 1000/day. High-volume labs may hit limits.

- Single host: All services run on one VM. Production SOCs distribute across multiple nodes.

-No automatic remediation: The pipeline notifies but does not isolate endpoints or block IPs beyond Wazuh's Active Response.

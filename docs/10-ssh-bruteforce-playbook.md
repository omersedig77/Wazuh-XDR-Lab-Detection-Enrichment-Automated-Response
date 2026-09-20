# 10 - SSH Brute Force Enrichment Playbook

## Overview

This playbook enriches SSH brute-force alerts with threat intelligence from VirusTotal and AbuseIPDB, then routes them to Discord with severity-based formatting.

## Trigger

- **Primary Rule:** `5763` — `sshd: brute force trying to get access`
- **Additional Matching Rules:** `5760`, `5758`, `2502`, `5551`, `40111`
- **Level Threshold:** `7` or higher

## Workflow Diagram

```text
Wazuh Alert
    │
    ▼
Extract Fields
    │
    ▼
If: rule_id in brute-force list?
    │
    ├── TRUE ──► VT Lookup (IP)
    │                │
    │                ▼
    │           AbuseIPDB Lookup
    │                │
    │                ▼
    │          Build Enriched Alert
    │                │
    │                ▼
    │           Severity Router
    │                │
    │                ├── High Risk ──► Discord High Risk
    │                ├── Suspicious ─► Discord Suspicious
    │                ├── Internal ───► Discord Internal
    │                └── Fallback ───► (unused)
    │
    └── FALSE ─► (no action)
```

## Node Configuration

### Wazuh Alert (Webhook)

| Field | Value |
|---|---|
| HTTP Method	| POST |
| Path	| wazuh-alerts |
| Authentication |	None |
| Response Mode	| Immediately

### Extract Fields (Set)

| Field Name |	Type |	Value |
|---|---|---|
| alert_title |	String |	{{ $json.body.rule.description }} |
| alert_level |	Number |	{{ $json.body.rule.level }} |
| agent_name |	String |	{{ $json.body.agent.name }} |
| source_ip |	String |	{{ $json.body.data.srcip }} |
| rule_id |	String |	{{ $json.body.rule.id }} |

### If (Conditional)

Condition: ```alert_level >= 5```

Convert types where required: ON

### VT Lookup (HTTP Request)

|Field | Value |
|---|---|
| Method |	GET |
|URL |https://www.virustotal.com/api/v3/ip_addresses/{{ $('Extract Fields').item.json.source_ip }} |
|Authentication |	Generic Credential Type → Header Auth |
|Credential |	VirusTotal API (Name: x-apikey) |
|Ignore SSL Issues |	ON |
|Response Format |	JSON |

### AbuseIPDB Lookup (HTTP Request)

| Field |	Value |
|---|---|
| Method |	GET |
| URL | https://api.abuseipdb.com/api/v2/check?ipAddress={{ $('Extract Fields').item.json.source_ip }}&maxAgeInDays=90 |
| Authentication | Generic Credential Type → Header Auth |
| Credential | AbuseIPDB API (Name: Key) |
| Headers | Accept: application/json |
| Ignore SSL Issues |	ON |
| Response Format | JSON |

### Build Enriched Alert (Set)

| Field Name |	Type |	Value |
|---|---|---|
| alert_title |	String |	{{ $('Extract Fields').item.json.alert_title }} |
| rule_id |	String |	{{ $('Extract Fields').item.json.rule_id }} |
| severity |	Number |	{{ $('Extract Fields').item.json.alert_level }} |
| agent_name |	String |	{{ $('Extract Fields').item.json.agent_name }} |
| source_ip |	String |	{{ $('Extract Fields').item.json.source_ip }} |
| vt_malicious |	Number |	{{ $('VT Lookup').item.json.data.attributes.last_analysis_stats.malicious }} |
| vt_suspicious |	Number |	{{ $('VT Lookup').item.json.data.attributes.last_analysis_stats.suspicious }} |
| vt_reputation |	Number |	{{ $('VT Lookup').item.json.data.attributes.reputation }} |
| vt_country |	String |	{{ $('VT Lookup').item.json.data.attributes.country }} |
| vt_as_owner |	String |	{{ $('VT Lookup').item.json.data.attributes.as_owner }} |
| abuse_score |	Number |	{{ $json.data.abuseConfidenceScore }} |
| abuse_reports |	Number |	{{ $json.data.totalReports }} |
| abuse_usage |	String |	{{ $json.data.usageType }} |
| abuse_domain |	String |	{{ $json.data.domain }} |
| abuse_country |	String |	{{ $json.data.countryCode }} |

### Severity Router (Switch)

Rule 1 - High Risk:

Condition: {{ $json.abuse_score }} greater than 50

Output: High Risk

Rule 2 - Suspicious:

Condition: {{ $json.vt_malicious }} greater than 3

Output: Suspicious

Rule 3 - Internal:

Condition: {{ $json.source_ip }} starts with 192.168.50

Output: Internal

Fallback: for unmatched alerts

Discord Nodes (HTTP Request)
Three HTTP Request nodes, one per severity branch. Each sends a rich embed to Discord.

High Risk (Red, color 15158332):

🚨 {alert_title}

HIGH RISK — External attacker with malicious reputation.

Rule ID: {rule_id}
Severity: {severity}
Agent: {agent_name}
Source IP: {source_ip}

VirusTotal AbuseIPDB
Malicious: {vt_malicious} Confidence: {abuse_score}%
Suspicious: {vt_suspicious} Reports: {abuse_reports}
Reputation: {vt_reputation} Usage: {abuse_usage}
Country: {vt_country} Domain: {abuse_domain}
AS Owner: {vt_as_owner} Country: {abuse_country}

Wazuh XDR Lab — SOC L1 SOAR | Priority: HIGH

Suspicious (Orange, color 15105570):

Same layout, but:

Content: 🟠 SUSPICIOUS SSH BRUTE FORCE

Description prefix: SUSPICIOUS — Moderate reputation concern

Footer: Priority: MEDIUM

Internal (Yellow, color 16776960):

Same layout, but:

Content: 🟡 INTERNAL SSH BRUTE FORCE

Description prefix: INTERNAL — Source IP is on the internal network. Likely lab activity.

Footer: Priority: LOW

Testing
Manual Test via curl
bash
curl -X POST "http://192.168.159.138:5678/webhook/wazuh-alerts" \
  -H "Content-Type: application/json" \
  -d '{
    "rule": {"id": "5763", "level": 10, "description": "sshd: brute force trying to get access"},
    "agent": {"name": "Linux-Endpoint", "ip": "192.168.50.30"},
    "data": {"srcip": "192.168.50.40"}
  }'
Real Attack Test from Kali
bash
for i in $(seq 1 25); do
  sshpass -p "wrong$i" ssh -o StrictHostKeyChecking=no -o ConnectTimeout=2 root@192.168.50.30 2>/dev/null
done
Expected Result
A Discord embed appears within 5 seconds with:

Alert title

Enriched VirusTotal data (malicious, suspicious, reputation, country, AS owner)

Enriched AbuseIPDB data (confidence, reports, usage, domain, country)

Color matches severity

MITRE ATT&CK Mapping
Technique	ID
Brute Force: Password Guessing	T1110.001
Brute Force: Password Cracking	T1110.002
Evidence
See screenshots/08-soar/ for Discord alert examples and workflow canvas.

Troubleshooting
Issue	Fix
Webhook returns 404	Workflow must be Active (production mode)
VT Lookup fails with 401	Check VirusTotal API credential
AbuseIPDB fails with 403	Check AbuseIPDB API credential
If node always routes FALSE	Verify rule_id matches incoming alert
Discord shows red "sendLegacy" error	Use HTTP Request node instead of Discord node

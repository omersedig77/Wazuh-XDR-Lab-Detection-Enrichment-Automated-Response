# 12 - Phishing Triage Playbook

## Overview

This playbook automates the triage of reported phishing emails. It enriches the sender, sender domain, and embedded URLs with multiple threat intelligence sources, applies a verdict, and delivers an actionable alert to Discord.

Phishing triage is the most common L1 SOC task. This playbook demonstrates end-to-end automation of that workflow.

## Trigger

A phishing email is submitted via a custom webhook. In this lab, the payload is simulated with curl. In production, this would be fed by:
- A user-reported email mailbox (Microsoft 365 / Google Workspace)
- An email security gateway (Proofpoint, Mimecast, etc.)
- A phishing report button plugin

Webhook endpoint: `POST http://192.168.159.138:5678/webhook/phishing-intake`

## Workflow Diagram

    Phishing Intake (Webhook)
         │
         ▼
    Parse Email (Set)
         │
         ▼
    urlscan.io Submit (HTTP)
         │
         ▼
    AbuseIPDB Lookup (HTTP)
         │
         ▼
    Extract Domain (Set)
         │
         ▼
    VT Domain Lookup (HTTP)
         │
         ▼
    Build Enriched Alert (Set)
         │
         ▼
    Severity Router (Switch)
         │
         ├─ Malicious ────────► Discord Malicious
         ├─ Suspicious ───────► Discord Suspicious
         ├─ Suspicious Domain ─► Discord Suspicious
         └─ Fallback ─────────► Discord Clean

## Expected Input

The webhook expects a JSON payload with the following structure:

```json
{
  "sender": "attacker@malicious-domain.com",
  "reply_to": "reply@another-suspicious.com",
  "subject": "Urgent: Verify Your Account",
  "recipient": "user@company.com",
  "sender_ip": "8.8.8.8",
  "urls": ["http://example.com"],
  "attachment": null,
  "attachment_hash": ""
}
```
## Node Configuration

### Phishing Intake (Webhook)
|Field |	Value |
|---|---|
| HTTP Method |	POST |
| Path |	phishing-intake |
| Authentication |	None |
| Response Mode |	Immediately |

### Parse Email (Set)
| Field Name |	Type |	Value |
|---|---|---|
| sender |	String |	{{ $json.body.sender }} |
| reply_to |	String |	{{ $json.body.reply_to }} |
| subject |	String |	{{ $json.body.subject }} |
| recipient |	String |	{{ $json.body.recipient }} |
| sender_ip |	String |	{{ $json.body.sender_ip }} |
| first_url |	String |	{{ $json.body.urls[0] }} |
| url_count |	Number |	{{ $json.body.urls.length }} |
| has_attachment |	Boolean |	{{ $json.body.attachment ? true : false }} |
| attachment_hash |	String |	{{ $json.body.attachment_hash		"" }} |

### urlscan.io Submit (HTTP Request)
| Field |	Value |
|---|---|
| Method |	POST |
| URL |	https://urlscan.io/api/v1/scan/ |
| Authentication |	Generic Credential Type → Header Auth |
| Credential |	urlscan.io API (Name: api-key) |
| Send Headers |	ON |
| Headers |	Content-Type: application/json |
| Send Body |	ON |
| Body Content Type |	JSON |
| JSON |	{"url": "{{ $json.first_url }}", "visibility": "public"} |
| Continue On Fail |	ON |
| Ignore SSL Issues |	ON |
| Response Format |	JSON |

Note: urlscan.io deduplicates URLs. If a URL was scanned recently, urlscan returns a 400 error. "Continue On Fail" prevents this from breaking the pipeline.

### AbuseIPDB Lookup (HTTP Request)
| Field |	Value |
|---|---|
| Method |	GET |
| URL |	https://api.abuseipdb.com/api/v2/check?ipAddress={{ $('Parse Email').item.json.sender_ip }}&maxAgeInDays=90 |
| Authentication |	Generic Credential Type → Header Auth |
| Credential |	AbuseIPDB API (Name: Key) |
| Send Headers |	ON |
| Headers |	Accept: application/json |
| Ignore SSL Issues |	ON |
| Response Format |	JSON |

### Extract Domain (Set)
| Field Name |	Type |	Value |
|---|---|---|
| sender_domain |	String |	{{ $('Parse Email').item.json.sender.split('@')[1] }} |

### VT Domain Lookup (HTTP Request)
| Field |	Value |
|---|---|
| Method |	GET |
| URL |	https://www.virustotal.com/api/v3/domains/{{ $json.sender_domain }} |
| Authentication |	Generic Credential Type → Header Auth |
| Credential |	VirusTotal API (Name: x-apikey) |
| Ignore SSL Issues |	ON |
| Response Format |	JSON |

### Build Enriched Alert (Set)
| Field Name |	Type |	Value |
|---|---|---|
| sender |	String |	{{ $('Parse Email').item.json.sender }} |
| reply_to |	String |	{{ $('Parse Email').item.json.reply_to }} |
| subject |	String |	{{ $('Parse Email').item.json.subject }} |
| recipient |	String |	{{ $('Parse Email').item.json.recipient }} |
| sender_ip |	String |	{{ $('Parse Email').item.json.sender_ip }} |
| sender_domain |	String |	{{ $('Extract Domain').item.json.sender_domain }} |
| first_url |	String |	{{ $('Parse Email').item.json.first_url }} |
| url_count |	Number |	{{ $('Parse Email').item.json.url_count }} |
| has_attachment |	Boolean |	{{ $('Parse Email').item.json.has_attachment }} |
| attachment_hash |	String |	{{ $('Parse Email').item.json.attachment_hash }} |
| urlscan_uuid |	String |	{{ $('urlscan.io Submit').item.json.uuid		'N/A' }} |
| urlscan_result_link |	String | {{ ('urlscan.ioSubmit').item.json.uuid?'https: //urlscan.io/result/'+('urlscan.ioSubmit').item.json.uuid + '/' : 'N/A' }} |
| abuse_score |	Number |	{{ $('AbuseIPDB Lookup').item.json.data.abuseConfidenceScore }} |
| abuse_reports |	Number |	{{ $('AbuseIPDB Lookup').item.json.data.totalReports }} |
| abuse_country | String |	{{ $('AbuseIPDB Lookup').item.json.data.countryCode }} |
| abuse_usage |	String |	{{ $('AbuseIPDB Lookup').item.json.data.usageType }} |
| vt_domain_malicious |	Number |	{{ $json.data.attributes.last_analysis_stats.malicious }} |
| vt_domain_suspicious |	Number |	{{ $json.data.attributes.last_analysis_stats.suspicious }} |
| vt_domain_reputation |	Number |	{{ $json.data.attributes.reputation }} |

### Severity Router (Switch)
Rule 1 - Malicious:

Condition: {{ $json.abuse_score }} greater than or equal to 50

Output: Malicious

Rule 2 - Suspicious:

Condition: {{ $json.vt_domain_malicious }} greater than or equal to 3

Output: Suspicious

Rule 3 - Suspicious Domain:

Condition: {{ $json.vt_domain_suspicious }} greater than or equal to 2

Output: Suspicious

Fallback: Clean

Note: Rules 2 and 3 both route to the same Discord output (Discord Suspicious). This is intentional — the analyst cares about the verdict (Suspicious), not which specific rule triggered.

Discord Malicious (HTTP Request)
Field	Value
Method	POST
URL	(Discord webhook URL)
Headers	Content-Type: application/json
Body	JSON
json
{
  "content": "🎣 **PHISHING EMAIL DETECTED**",
  "embeds": [
    {
      "title": "🚨 Phishing Triage: {{ $json.subject }}",
      "description": "**MALICIOUS** — Email classified as phishing.\n\n**From:** {{ $json.sender }}\n**Reply-To:** {{ $json.reply_to }}\n**To:** {{ $json.recipient }}",
      "color": 15158332,
      "fields": [
        {
          "name": "🌍 Sender Reputation",
          "value": "IP: {{ $json.sender_ip }}\nDomain: {{ $json.sender_domain }}\nAbuse Score: **{{ $json.abuse_score }}%**\nCountry: {{ $json.abuse_country }}",
          "inline": true
        },
        {
          "name": "🔗 URLs",
          "value": "Count: {{ $json.url_count }}\nFirst URL: `{{ $json.first_url }}`\nurlscan: {{ $json.urlscan_result_link }}",
          "inline": false
        },
        {
          "name": "📎 Attachment",
          "value": "{{ $json.has_attachment ? 'Present — Hash: ' + $json.attachment_hash : 'None' }}",
          "inline": false
        }
      ],
      "footer": {
        "text": "Wazuh XDR Lab — SOC L1 SOAR | Action: QUARANTINE + BLOCK SENDER"
      }
    }
  ]
}
Discord Suspicious (HTTP Request)
Same structure as Discord Malicious, with these changes:

Content: 🟠 **SUSPICIOUS EMAIL DETECTED**

Color: 15105570

Description prefix: **SUSPICIOUS** — Email shows phishing indicators.

Footer: Priority: MEDIUM | Action: Manual review

Discord Clean (HTTP Request)
Same structure, with:

Content: 🟢 **EMAIL OBSERVED (Clean)**

Color: 3066993

Description prefix: **CLEAN** — No phishing indicators found.

Footer: Priority: LOW

Testing
Manual Test via curl
bash
curl -X POST "http://192.168.159.138:5678/webhook/phishing-intake" \
  -H "Content-Type: application/json" \
  -d '{
    "sender": "attacker@malicious-domain.com",
    "reply_to": "reply@another-suspicious.com",
    "subject": "Urgent: Verify Your Account",
    "recipient": "user@company.com",
    "sender_ip": "8.8.8.8",
    "urls": ["http://example.com"],
    "attachment": null,
    "attachment_hash": ""
  }'
Expected Result
Discord receives a phishing triage alert within 5 seconds, classified as either Malicious, Suspicious, or Clean based on enrichment.

The alert includes:

Subject

Sender / Reply-To / Recipient

Sender reputation (IP + domain)

Embedded URL details

urlscan.io result link

Attachment status

Recommended action

MITRE ATT&CK Mapping
Technique	ID
Phishing: Spearphishing Attachment	T1566.001
Phishing: Spearphishing Link	T1566.002
Known Limitations
urlscan.io deduplication: Submitting the same URL twice returns a 400. The "Continue On Fail" setting prevents this from breaking the pipeline.

No attachment analysis: Attachment hashes are recorded but not enriched. Future work: query VT file endpoint.

No reply-to vs sender mismatch check: A key phishing indicator. Future work: compare domains.

No SPF/DKIM/DMARC validation: Would require email headers. Future work.

Evidence
See screenshots/09-phishing/ for Discord alert and workflow canvas.

Troubleshooting
Issue	Fix
urlscan.io returns 400	URL already scanned (dedup) — Continue On Fail handles it
urlscan.io returns 401	Check api-key credential
VT Domain Lookup returns 404	Domain not in VT database (common for new domains)
Wrong verdict	Adjust thresholds in Severity Router
Discord silent	Check workflow is Active

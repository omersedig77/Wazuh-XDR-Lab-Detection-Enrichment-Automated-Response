# L1 Triage Report — Alert 003: Phishing Email Reported

| Field | Value |
|---|---|
| **Alert ID** | ALT-003 |
| **Date/Time** | 2026-09-20 16:53 UTC |
| **Analyst** | SOC L1 |
| **Severity** | Medium |
| **Source** | User-submitted phishing report (webhook) |
| **Sender** | attacker@malicious-domain.com |
| **Subject** | Urgent: Verify Your Account |
| **Verdict** | Suspicious |
| **MITRE** | T1566.002 (Phishing: Spearphishing Link) |

---

## 1. Alert Summary

A user reported a suspicious email through the phishing report webhook. The SOAR pipeline automatically enriched the email and classified it as **Suspicious** based on sender reputation and URL analysis.

## 2. Alert Details

| Field | Value |
|---|---|
| Sender | attacker@malicious-domain.com |
| Reply-To | reply@another-suspicious.com |
| Recipient | user@company.com |
| Subject | Urgent: Verify Your Account |
| Sender IP | 8.8.8.8 |
| Sender Domain | malicious-domain.com |
| URLs Count | 1 |
| First URL | http://example.com |
| Attachment | None |
| Attachment Hash | (empty) |

## 3. Enrichment Results

### Sender IP Reputation (AbuseIPDB)

| Field | Value |
|---|---|
| Abuse Confidence | 0% |
| Reports | 13 |
| Country | US |
| Usage Type | Reserved |

### Sender Domain Reputation (VirusTotal)

| Field | Value |
|---|---|
| Malicious Detections | Not in database |
| Suspicious Detections | N/A |
| Domain Reputation | N/A (new domain) |

### URL Analysis (urlscan.io)

| Field | Value |
|---|---|
| Scan UUID | 017a0be1-155c-7071-a65f-66cc6ea4c79e |
| Result Link | https://urlscan.io/result/017a0be1.../ |

## 4. Investigation Steps

1. Reviewed email headers and content
2. Enriched sender IP via AbuseIPDB — no abuse reports, "Reserved" usage (likely cloud infra)
3. Extracted sender domain — submitted to VirusTotal — no existing record (new domain)
4. Submitted URL to urlscan.io — scan completed successfully
5. Reviewed Reply-To vs From mismatch — different domains (red flag)
6. Analyzed subject line — urgency trigger ("Urgent")
7. Cross-referenced sender domain against known phishing databases — no hits

## 5. Evidence

- SOAR alert: `screenshots/09-phishing/01-discord-alert.png`
- urlscan.io report: https://urlscan.io/result/017a0be1.../
- Email JSON payload: `evidence/alert-003-email.json`

## 6. Verdict

**Suspicious** — Multiple phishing indicators present:
- **Reply-To mismatch**: `reply@another-suspicious.com` ≠ `attacker@malicious-domain.com`
- **Urgency subject**: "Urgent: Verify Your Account"
- **New sender domain**: No reputation history in VT
- **Unknown URL**: Submitted for scanning

However, no confirmed malicious detections on:
- Sender IP (0% abuse confidence)
- Sender domain (no VT record)
- URL (pending urlscan analysis)

**Classification:** Suspicious (require human review)

## 7. Action Taken

- **SOAR Playbook executed:** Phishing Triage Playbook
- **Notification sent:** Discord SOC Alerts channel (Priority: MEDIUM)
- **Recommended action:** Manual review before blocking
- **User informed:** Standard phishing awareness guidance

## 8. Escalation

**Escalated to L2** — Suspicious emails that pass initial automated checks need human review for:
- Visual inspection of email body (screenshots, branding, grammar)
- Analysis of any embedded tracking pixels
- Correlation with other user reports
- Confirmation of URL destination by sandbox detonation

## 9. Recommendations

- Add SPF/DKIM/DMARC verification to the playbook
- Implement Reply-To domain mismatch as automatic Suspicious trigger
- Add attachment hash VT lookup (currently recorded but not enriched)
- Integrate with mail gateway for automated quarantine recommendation

## 10. MITRE ATT&CK Mapping

| Tactic | Technique | ID |
|---|---|---|
| Initial Access | Phishing: Spearphishing Link | T1566.002 |

---

**Status:** Escalated to L2 — Suspicious
**SOC L1 Analyst:** [Omer]

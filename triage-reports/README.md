# L1 Triage Report Library

This folder contains professional SOC L1 triage reports written for real alerts generated in the lab environment.

Each report follows a consistent structure that mirrors how an L1 analyst documents alerts in a real SOC:

1. **Alert Summary** — What fired and why
2. **Alert Details** — Raw evidence
3. **Initial Triage** — Quick assessment questions
4. **Investigation Steps** — What the analyst did
5. **Evidence** — Screenshots, logs, links
6. **Verdict** — True Positive / False Positive / Suspicious
7. **Action Taken** — Response and notification
8. **Escalation** — Whether to raise to L2 and why
9. **Recommendations** — Improvement suggestions
10. **MITRE ATT&CK Mapping** — Technique association

## Report Index

| # | Report | Verdict | Severity |
|---|---|---|---|
| 001 | [SSH Brute Force](alert-001-ssh-bruteforce.md) | True Positive | High |
| 002 | [Malware File Detected](alert-002-malware-detection.md) | True Positive (Test) | Critical |
| 003 | [Phishing Email Reported](alert-003-phishing-email.md) | Suspicious | Medium |
| 004 | [Active Response Auto-Block](alert-004-active-response-block.md) | True Positive | High |
| 005 | [Critical CVE Detected](alert-005-vulnerability-cve.md) | True Positive | Critical |

## Why This Matters

L1 analysts spend most of their day writing triage reports. This library demonstrates:

- Consistent documentation standards
- Clear reasoning for verdicts
- Professional escalation criteria
- Evidence-based investigation
- MITRE ATT&CK mapping
- Actionable recommendations

This is exactly what hiring managers look for in a junior SOC candidate.

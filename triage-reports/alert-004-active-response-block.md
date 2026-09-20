# L1 Triage Report — Alert 004: Active Response Auto-Block

| Field | Value |
|---|---|
| **Alert ID** | ALT-004 |
| **Date/Time** | 2026-09-13 14:52 UTC |
| **Analyst** | SOC L1 |
| **Severity** | High |
| **Rule ID** | 5763, 601 |
| **Rule Description** | SSH brute force detected + Host blocked by firewall-drop |
| **Agent** | Linux-Endpoint (192.168.50.30) |
| **Source IP** | 192.168.50.40 (Kali) |
| **MITRE** | T1110.001 (Brute Force: Password Guessing) |

---

## 1. Alert Summary

Wazuh detected an SSH brute-force attack from `192.168.50.40` against `192.168.50.30`. The Active Response module automatically blocked the source IP using `iptables` for 180 seconds.

## 2. Alert Details

| Field | Value |
|---|---|
| Attack Type | SSH Brute Force |
| Source IP | 192.168.50.40 |
| Target Endpoint | Linux-Endpoint (192.168.50.30) |
| Target User | root |
| Attempts | 16+ within 30 seconds |
| Successful Login | No |
| Active Response | firewall-drop (iptables) |

## 3. Investigation Steps

1. Reviewed Wazuh alert in Threat Hunting dashboard
2. Confirmed source IP `192.168.50.40` is the Kali attacker VM (internal)
3. Reviewed `/var/log/auth.log` on the endpoint — repeated failed password attempts
4. Verified Active Response fired: `iptables` DROP rule applied
5. Tested connectivity from Kali to endpoint — 100% packet loss
6. Waited 180 seconds — iptables rule removed automatically
7. Tested connectivity again — restored

## 4. Evidence

- Alert log: rules 5763, 601
- `iptables -L INPUT -n` output showing DROP rule for 192.168.50.40
- Kali ping output: 100% packet loss during block
- Screenshots: `screenshots/04-active-response/`

## 5. Verdict

**True Positive** — Confirmed SSH brute-force attack. Source IP is a known lab attacker (Kali). Active Response contained it automatically.

**Classification:** True Positive (contained)

## 6. Action Taken

- **Automated containment:** Active Response blocked source IP for 180 seconds
- **SOAR Playbook executed:** SSH Brute Force Playbook delivered enriched Discord alert
- **Notification sent:** Discord SOC Alerts channel
- **No manual intervention required**

## 7. Escalation

**Not escalated** — The attack originated from an internal lab VM. Automated containment was sufficient. In production, an external source would be escalated to L2 for permanent block.

## 8. Recommendations

- Extend Active Response timeout to 600 seconds for repeat offenders
- Add persistent block list (fail2ban integration)
- Enable per-user rate limiting
- Configure Active Response for Windows endpoints (netsh)

## 9. MITRE ATT&CK Mapping

| Tactic | Technique | ID |
|---|---|---|
| Credential Access | Brute Force: Password Guessing | T1110.001 |

---

**Status:** Closed — True Positive (auto-contained)
**SOC L1 Analyst:** [Your Name]

# L1 Triage Report — Alert 001: SSH Brute Force

| Field | Value |
|---|---|
| **Alert ID** | ALT-001 |
| **Date/Time** | 2026-09-13 18:21 UTC |
| **Analyst** | [Your Name] |
| **Severity** | High |
| **Rule ID** | 5763 |
| **Rule Description** | sshd: brute force trying to get access |
| **Agent** | Linux-Endpoint (192.168.50.30) |
| **Source IP** | 192.168.50.40 |

---

## 1. Alert Summary

Wazuh detected a rapid sequence of failed SSH authentication attempts against the Linux endpoint (`192.168.50.30`) from the source IP `192.168.50.40`.

## 2. Initial Triage

| Question | Answer |
|---|---|
| Is the source IP internal or external? | Internal (192.168.50.40) |
| Is the target user a valid account? | `root` |
| Number of failed attempts? | 16+ within ~10 seconds |
| Was any login successful? | No |

## 3. Investigation Steps

1. Reviewed Wazuh alert details in **Threat Hunting** dashboard
2. Verified source IP `192.168.50.40` — matches known **Kali Linux** attacker VM
3. Checked target user — `root` (a privileged account)
4. Reviewed `/var/log/auth.log` on the Linux endpoint — confirmed repeated `Failed password for root` entries
5. Confirmed Active Response triggered: `iptables` DROP rule for `192.168.50.40`

## 4. Evidence

- Screenshot: `screenshots/04-active-response/iptables-drop-rule.png`
- Screenshot: `screenshots/04-active-response/kali-blocked.png`
- Wazuh alert log: rule 5763 + rule 601

## 5. Verdict

**True Positive** — Confirmed SSH brute-force attack simulating a real adversary technique (MITRE T1110.001).

## 6. Action Taken

- ✅ **Automated Response executed:** `firewall-drop` blocked `192.168.50.40` for 180 seconds
- ✅ **Confirmed block:** Kali was unable to reach `192.168.50.30` (`100% packet loss`)
- ⏳ **Analyst action:** Document incident, close ticket after auto-unblock

## 7. Escalation

**Not escalated** — the attack originated from a known internal lab VM. In a production environment, this would be **escalated to L2** for:
- Verifying whether the account was compromised
- Checking for lateral movement from the source
- Blocking the IP permanently at the perimeter

## 8. Recommendations

- ✅ Continue using Active Response for this rule
- ⏳ Enable VirusTotal enrichment on the source IP (future enhancement)
- ⏳ Correlate with pfSense firewall logs for full visibility

## 9. MITRE ATT&CK Mapping

| Tactic | Technique | ID |
|---|---|---|
| Credential Access | Brute Force: Password Guessing | T1110.001 |

---

**Status:** Closed — True Positive (Contained)

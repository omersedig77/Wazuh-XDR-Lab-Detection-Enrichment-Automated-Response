# Detection: SSH Brute Force

## Overview

Detects repeated failed SSH authentication attempts against a Linux endpoint — a classic brute-force / password-guessing pattern.

## MITRE ATT&CK

| Technique | ID |
|---|---|
| Brute Force: Password Guessing | T1110.001 |
| Brute Force: Password Cracking | T1110.002 |

## Rules

| Rule ID | Level | Description |
|---|---|---|
| 5710 | 5 | sshd: Attempt to login using a non-existent user |
| 5716 | 8 | sshd: insecure connection attempt |
| 5720 | 8 | sshd: Multiple authentication failures |
| **5763** | **10** | **sshd: brute force trying to get access** |

Rule 5763 is the primary indicator — it fires when multiple authentication failures originate from the same source IP within a short window.

## Data Source

- `/var/log/auth.log` (Linux endpoint)
- Wazuh decoder: `sshd`
- Monitored by: Wazuh agent on Linux endpoint

## Example Alert

```json
{
  "rule": {
    "id": "5763",
    "level": 10,
    "description": "sshd: brute force trying to get access"
  },
  "data": {
    "srcip": "192.168.50.40",
    "dstuser": "root"
  },
  "agent": {
    "name": "Linux-Endpoint",
    "ip": "192.168.50.30"
  }
}
```

## Response
Active Response (firewall-drop) blocks the source IP for 180 seconds.

## Tuning Notes
The threshold for rule 5763 is defined in Wazuh's default ruleset (0095-sshd_rules.xml).

Increase frequency if you see false positives from legitimate users mistyping passwords.

## References

- Wazuh SSH rules documentation

- MITRE ATT&CK T1110

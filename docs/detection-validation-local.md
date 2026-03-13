# Detection Validation — Local Wazuh SIEM

## Objective
Validate that the local Wazuh deployment correctly detects, alerts, and
correlates security-relevant activity on the ARM64 homeserver endpoint.

## Environment
- **Wazuh Manager:** wazuh-server (x86_64 UTM VM, 192.168.64.5)
- **Wazuh Version:** 4.7.5
- **Endpoint Agent:** homeserver (Ubuntu 24.04 ARM64, agent ID 001)
- **Log Sources:** auth.log, PAM, sshd, sudo events
- **Transport:** TCP 1514 (telemetry), TCP 1515 (enrollment)

---

## Test 1 — SSH Brute Force Detection

### Actions performed

```bash
# Repeated failed SSH login attempts with non-existent user
ssh fakeuser@192.168.64.2  # x5

# Legitimate login
ssh user@192.168.64.2
```

### Detections observed

| Rule ID | Description | Level |
|---|---|---|
| 5710 | sshd: Attempt to login using a non-existent user | 5 |
| 5503 | PAM: User login failed | 5 |
| 2502 | syslog: User missed the password more than one time | 10 |
| 40101 | System user successfully logged to the system | 12 |
| 5501 | PAM: Login session opened | 3 |

### Summary
- 24 authentication failures recorded
- Brute force behaviour correctly escalated to level 10
- Successful login after failures triggered level 12 alert
- Legitimate and malicious events correctly differentiated

---

## Test 2 — Privilege Escalation Detection

### Actions performed

```bash
sudo su
whoami
id
exit
```

### Detections observed

| Rule ID | Description | Level | MITRE |
|---|---|---|---|
| 5402 | Successful sudo to ROOT executed | 3 | T1548.003 |
| 5501 | PAM: Login session opened | 3 | T1078 |

### Summary
- Privilege escalation via sudo correctly detected
- MITRE ATT&CK mappings applied automatically
- Subsequent commands (whoami, id) correlated to same elevated session
- No duplicate alerts generated — correct noise reduction behaviour

---

## Dashboard Observations

- **Total alerts (15 min window):** 29
- **Level 12+ alerts:** 1
- **Authentication failures:** 24
- **Authentication successes:** 3
- **Top rule groups:** authentication_failed, syslog, invalid_login, sshd, pam

Alert groups evolution graph showed a clear spike at time of simulated attack,
demonstrating real-time detection capability.

---

## MITRE ATT&CK Mapping

Wazuh automatically mapped all detections to the MITRE ATT&CK framework.

### Tactics observed

| Tactic | Techniques |
|---|---|
| Credential Access | Password Guessing, SSH |
| Lateral Movement | SSH |
| Defense Evasion | Valid Accounts |
| Initial Access | Valid Accounts |
| Persistence | Valid Accounts |
| Privilege Escalation | Sudo and Sudo Caching |

Credential Access was the dominant tactic (largest volume), driven by the
SSH brute force simulation. Privilege Escalation was correctly isolated as
a separate tactic from the sudo activity.

---

## Key Takeaways

- Wazuh correctly differentiates malicious authentication attempts from
  normal user activity
- Brute force behaviour is detected through event correlation, not just
  individual failures
- Privilege escalation via sudo is detected and MITRE-mapped automatically
- Alert levels are proportionate — noise kept low, high-severity events
  correctly elevated
- Local deployment performs identically to the previous Azure-hosted setup

---

## Next Steps
- Add Suricata IDS for network-layer detection
- Write custom Wazuh rules for events not covered by defaults
- Expand attack simulations using Kali Linux

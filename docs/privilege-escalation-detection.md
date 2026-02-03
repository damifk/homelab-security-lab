# Detection with Wazuh

## Objective
Validate Wazuh’s ability to detect and log activity on a Linux endpoint using real system events, MITRE ATT&CK mapping, and SIEM alerting.

---

## Environment
- **Wazuh Manager:** Azure VM (Ubuntu)
- **Wazuh Version:** 4.7.x
- **Endpoint Agent:** Ubuntu Linux (ARM64)
- **Log Sources:** auth.log, PAM, sudo events
- **Transport:** TCP 1514 (agent data), TCP 1515 (agent enrollment)

---

## Actions Performed on Endpoint

### Step 16 – SSH Authentication and Brute Force Detection

To validate SSH monitoring and authentication detection, both invalid and
valid login attempts were performed on the monitored Linux host.

#### Simulated attacker activity
```bash
ssh fakeuser@localhost
```
#### Observed detections:
- SSH login attempts using a non-existent user
- PAM authentication failures
- Brute-force behaviour detection
- Multiple correlated alerts over a short time window

#### Example alerts:
- sshd: Attempt to login using a non-existent user
- PAM: User login failed
- sshd: brute force trying to get access to the system

These events demonstrate Wazuh’s ability to detect and escalate
unauthorised access attempts.

#### Legitimate user authentication
``` bash
ssh user@localhost
```

#### Observed detections:

- System user successfully logged to the system
- PAM: Login session opened

Successful authentication events were logged at a lower severity
and correctly classified as legitimate access.

#### Takeaways:

- Wazuh successfully differentiates between:
- Malicious authentication attempts
- Normal user activity


```bash 
# privilege escalation monitoring
sudo su
whoami
id
exit
```

Detection behaviour observed

Wazuh successfully detected and processed all commands executed.
However, alerts were correlated and grouped by security context, not
displayed as one alert per command.

Observed detections included:

PAM session opened

    MITRE: T1078 – Valid Accounts

Successful sudo execution

    MITRE: T1548.003 – Abuse Elevation Control Mechanism

PAM session closed

Commands such as whoami and id were logged and associated with the
elevated session but did not generate standalone alerts, as they did not
represent a new security boundary crossing.


Wazuh correctly:
- Detected privilege escalation
- Correlated subsequent commands to the same session
- Avoided alert duplication
- Maintained contextual awareness of user activity

This demonstrates effective alert correlation and noise reduction,
which aligns with real-world SOC monitoring practices.

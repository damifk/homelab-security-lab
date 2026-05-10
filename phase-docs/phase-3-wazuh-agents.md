# Phase 3 — Wazuh Agent Deployment

## Objective
Enroll `ubuntu-sensor` and `windows-endpoint` as Wazuh agents reporting to `wazuh-server`.

---

## Agent 1 — ubuntu-sensor (Linux)

### Installation

```bash
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo gpg --dearmor -o /usr/share/keyrings/wazuh.gpg

echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | sudo tee /etc/apt/sources.list.d/wazuh.list

sudo apt update

sudo WAZUH_MANAGER="192.168.100.128" WAZUH_AGENT_NAME="ubuntu-sensor" apt install wazuh-agent=4.7.5-1 -y
```

### Version Pinning
The default apt repository installs the latest agent (4.14.5 at time of build). Wazuh requires the agent version to be equal to or lower than the manager version. Pinning to `4.7.5-1` is mandatory.

### Manual Config Fix
The environment variable injection did not write the manager IP during reinstall. Manual edit required:

```bash
sudo nano /var/ossec/etc/ossec.conf
# Find <address>MANAGER_IP</address>
# Replace with <address>192.168.100.128</address>
```

### Start Agent

```bash
sudo systemctl daemon-reload
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
```

---

## Agent 2 — windows-endpoint (Windows)

### Installation

Download version-matched MSI:
```
https://packages.wazuh.com/4.x/windows/wazuh-agent-4.7.5-1.msi
```

Install via PowerShell (Administrator):
```powershell
msiexec.exe /i wazuh-agent-4.7.5-1.msi /q WAZUH_MANAGER="192.168.100.128" WAZUH_AGENT_NAME="windows-endpoint"
```

### Manual Config Fix
Same issue as Linux agent — manager IP written as `0.0.0.0`. Manual edit:
```
C:\Program Files (x86)\ossec-agent\ossec.conf
# Find <address>0.0.0.0</address>
# Replace with <address>192.168.100.128</address>
```

### Start Agent
```powershell
NET STOP WazuhSvc
NET START WazuhSvc
```

---

## Result

Both agents active and visible in Wazuh dashboard:

![Agents Active](../screenshots/02-agents-active.png)

| ID | Name | IP | OS | Version | Status |
|---|---|---|---|---|---|
| 001 | ubuntu-sensor | 192.168.100.129 | Ubuntu 24.04.4 LTS | v4.7.5 | active |
| 002 | DESKTOP-7ON7981 | 192.168.100.131 | Windows 10 Pro 10.0.19045.3803 | v4.7.5 | active |

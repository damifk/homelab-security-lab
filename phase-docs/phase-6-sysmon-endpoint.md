# Phase 6 — Sysmon Endpoint Telemetry

## Objective
Deploy Sysmon on `windows-endpoint` and configure Wazuh to collect endpoint telemetry for Windows-specific MITRE ATT&CK coverage.

## Sysmon Installation

Download:
- Sysmon: `https://download.sysinternals.com/files/Sysmon.zip`
- Config: `https://raw.githubusercontent.com/olafhartong/sysmon-modular/master/sysmonconfig.xml`

Save both to `C:\Tools\`, extract Sysmon zip.

Install via PowerShell (Administrator):
```powershell
cd C:\Tools
.\Sysmon64.exe -accepteula -i sysmonconfig.xml
```

Verify:
```powershell
Get-Service Sysmon64
# Status: Running
```

### Config Choice: Olaf Hartong Modular
Olaf Hartong's modular config was chosen over SwiftOnSecurity for its detection engineering focus. It provides broader MITRE ATT&CK coverage and more aggressive logging of process creation, network connections, and registry modifications — appropriate for a detection lab where high fidelity is preferable to noise reduction.

## Wazuh Integration

Add to `C:\Program Files (x86)\ossec-agent\ossec.conf` before closing `</ossec_config>`:

```xml
<localfile>
  <location>Microsoft-Windows-Sysmon/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>
```

Restart agent:
```powershell
NET STOP WazuhSvc
NET START WazuhSvc
```

## Immediate Results

Running basic enumeration commands (`whoami`, `ipconfig`, `net user`) immediately produced MITRE-mapped alerts:

| Alert | MITRE ID | Tactic |
|---|---|---|
| A net.exe account discovery command was initiated | T1087 | Discovery |
| Discovery activity executed | T1087 | Discovery |
| PowerShell scripting file created | T1105/T1059 | Command and Control, Execution |
| CIS Benchmark compliance failures | — | Configuration assessment |

![Windows Sysmon Alerts](../screenshots/05-windows-sysmon-alerts.png)


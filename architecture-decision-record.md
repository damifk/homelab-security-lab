# Architecture Decision Record

This document records the key architectural decisions made during the evolution of this homelab, including the reasoning behind each pivot. Architectural decision-making is itself a skill — this record preserves that context.

---

## ADR-001: Initial Platform — macOS / UTM / Docker

**Date:** Early 2026  
**Status:** Superseded by ADR-002

### Context
The lab was initially built on a MacBook Air (Apple Silicon, M-series). The goal was to run a Wazuh SIEM stack alongside a Suricata IDS sensor and a vulnerable web application (OWASP Juice Shop).

### Decision
Deploy using UTM virtualisation on macOS with Docker for containerised services.

### Outcome
A Docker-based Wazuh stack was attempted first but failed due to the ARM64 architecture — Wazuh's official container images do not support ARM64 at the time of this build.

This led to a split architecture:
- **homeserver** (ARM64 Ubuntu 24.04, UTM): Ran Docker (Juice Shop, Pi-hole), Wazuh agent, Suricata 7.0.3
- **wazuh-server** (x86_64 emulated Ubuntu 22.04, UTM): Ran Wazuh 4.7.5 all-in-one stack

The x86_64 emulation was required because Wazuh's official all-in-one installer hard-requires x86_64. UTM emulation introduced significant performance overhead, with the Wazuh installer requiring the `-i` (ignore hardware checks) flag and a minimum of 6GB RAM allocated to the VM just to satisfy OpenSearch's memory requirements.

### Key Learnings
- Wazuh's all-in-one installer rejects ARM64 at the hardware check stage — emulation is the only workaround on Apple Silicon
- OpenSearch (Wazuh's indexer) drives the high RAM requirement; 4GB is insufficient under emulation
- `ossec.conf` supports only a single `<ossec_config>` block — `<localfile>` entries cannot be nested inside `<client>`
- Full Suricata service restart is required when adding new rule files — USR2 signal reload is insufficient

---

## ADR-002: Migration to Windows PC / VMware Workstation Pro

**Date:** May 2026  
**Status:** Current

### Context
After completing the initial Mac-based lab phases (Suricata integration, Juice Shop attack simulation, custom SQLi detection rules), the decision was made to rebuild the lab on a Windows PC to remove the ARM64 constraint entirely and take the opportunity to restructure the architecture.

### Decision
Rebuild the lab on a Windows PC (x86_64) using VMware Workstation Pro (free since Broadcom acquisition) with four VMs on an isolated VMnet2 NAT network.

### Rationale

| Factor | Mac/UTM | Windows/VMware |
|---|---|---|
| Architecture | ARM64 host, x86_64 emulation required | Native x86_64 throughout |
| Wazuh installer | Required `-i` flag, 6GB RAM minimum | Clean install, 4GB sufficient |
| Performance overhead | High (emulation tax) | None |
| Windows endpoint | Additional complexity | Native Windows 10 VM available |
| Hypervisor | UTM (macOS only, limited features) | VMware Workstation Pro (mature, feature-rich) |
| Cost | Free | Free (VMware now free for personal use) |

### New Architecture
Rather than the reactive two-VM split forced by ARM64 constraints, the Windows rebuild was designed intentionally as a four-VM lab:

- `wazuh-server` — Dedicated SIEM (Ubuntu 22.04, 4GB RAM)
- `ubuntu-sensor` — Linux sensor and attack target (Ubuntu 24.04, 2GB RAM)
- `kali-attacker` — Attack simulation (Kali Linux, 2GB RAM)
- `windows-endpoint` — Windows endpoint telemetry (Windows 10 Pro, 4GB RAM)

Total RAM allocation: 12GB across all VMs, leaving 4GB for the Windows host. VMs are run in groups of 3 depending on the task rather than all simultaneously.

### Key Improvement: Windows Endpoint Telemetry
The Mac build had the Windows endpoint as a future phase. In the Windows rebuild, Sysmon was installed before attack simulation, meaning Kali attacks now generate telemetry across both network (Suricata) and endpoint (Sysmon) simultaneously — a more realistic SOC detection scenario.

### Network Design
All VMs connected to VMnet2 (`192.168.100.0/24`, NAT). Host virtual adapter enabled on VMnet2 to allow dashboard access from the Windows host browser.

| VM | IP |
|---|---|
| wazuh-server | 192.168.100.128 |
| ubuntu-sensor | 192.168.100.129 |
| kali-attacker | 192.168.100.130 |
| windows-endpoint | 192.168.100.131 |

### Outcome
Clean Wazuh installation without flags or workarounds. All four VMs provisioned and operational. Multi-platform telemetry (Linux + Windows) feeding into a single Wazuh instance. Custom Suricata SQLi detection rules validated against live attack traffic.

---

## ADR-003: Sysmon Config — Olaf Hartong Modular vs SwiftOnSecurity

**Date:** May 2026  
**Status:** Current

### Decision
Use Olaf Hartong's modular `sysmonconfig.xml` rather than the SwiftOnSecurity config.

### Rationale
SwiftOnSecurity's config is widely used and well-maintained but is optimised for noise reduction in production environments. Olaf Hartong's modular config is purpose-built for detection engineering — it covers a broader range of MITRE ATT&CK techniques and is more aggressive in logging process creation, network connections, and registry modifications. For a detection lab, higher fidelity is preferable to lower noise.

### Outcome
Immediate MITRE-mapped alerts on basic commands (`net user`, `whoami`, `ipconfig`) including T1087 (Account Discovery) and T1548 (Abuse Elevation Control).

---

## ADR-004: Wazuh Agent Version Pinning

**Date:** May 2026  
**Status:** Current

### Context
When installing the Wazuh agent on `ubuntu-sensor`, the default apt repository installed Wazuh agent 4.14.5 — significantly newer than the manager version (4.7.5). The agent refused to connect with the error: `Agent version must be lower or equal to manager version`.

### Decision
Pin the agent installation to version 4.7.5-1 to match the manager.

### Command
```bash
sudo apt install wazuh-agent=4.7.5-1
```

### Lesson
Always match agent versions to the manager version when installing from Wazuh's apt repository, as the repo always points to the latest available agent. The same principle applies to the Windows MSI — download the version-specific MSI from `packages.wazuh.com/4.x/windows/wazuh-agent-4.7.5-1.msi`.

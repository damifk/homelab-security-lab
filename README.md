# Homelab Security Lab (SOC-Focused)

This repository documents my personal cybersecurity homelab designed to simulate
a real-world Security Operations Centre (SOC) environment.

The goal of this lab is to build practical experience in:
- Detection engineering
- Endpoint telemetry and agent management
- Local SIEM deployment
- Incident visibility and troubleshooting

The lab uses a **fully local architecture** combining an ARM64 services host
with a dedicated x86_64 SIEM VM, both running under UTM on Apple Silicon.
Azure is reserved exclusively for Microsoft Sentinel and Defender practice.

---

## High-Level Architecture

```
[Local Lab - Apple Silicon (UTM)]
│
├─ Ubuntu Server ARM64 (homeserver)
│   ├─ Docker & Docker Compose
│   ├─ OWASP Juice Shop (port 8081)
│   ├─ Pi-hole (port 8080)
│   ├─ Suricata IDS 7.0.3 (enp0s1)
│   └─ Wazuh Agent (4.7.5)
│
├─ Ubuntu Server x86_64 (wazuh-server) [UTM Emulation]
│   ├─ Wazuh Manager (4.7.5)
│   ├─ Wazuh Indexer (OpenSearch)
│   ├─ Wazuh Dashboard (port 443)
│   └─ Filebeat
│
└─ Kali Linux ARM64
    └─ Attack simulation
```

---

## Environment

### ARM64 Services Host (homeserver)
- Virtualisation: UTM (Apple Hypervisor)
- OS: Ubuntu Server 22.04 ARM64
- RAM: 6GB
- Storage: 30GB
- Services: Docker, Juice Shop, Pi-hole, Wazuh Agent

### x86_64 SIEM Host (wazuh-server)
- Virtualisation: UTM (Emulation)
- OS: Ubuntu Server 22.04 x86_64
- RAM: 6GB
- Storage: 40GB
- Services: Wazuh Manager, Indexer, Dashboard

### Azure (Reserved)
- Platform: Microsoft Azure
- Use: Microsoft Sentinel, Defender for Cloud, Log Analytics
- Not used for Wazuh

---

## Completed Work

### Lab Architecture
- Confirmed Wazuh official installer is x86_64 only (hard architecture check)
- Confirmed Docker-based Wazuh is unstable on ARM64
- Provisioned dedicated x86_64 UTM emulation VM for SIEM
- Moved SIEM off Azure to preserve credits for Sentinel work

### ARM64 Host (homeserver)
- Installed Docker and Docker Compose
- Deployed OWASP Juice Shop (port 8081)
- Deployed Pi-hole DNS sinkhole (port 8080)
- Resolved port conflicts with systemd-resolved
- Removed broken Docker-based Wazuh stack

### Wazuh SIEM (wazuh-server - local x86_64)
- Provisioned x86_64 UTM VM
- Installed Wazuh 4.7.5 using official all-in-one installer
- Resolved indexer startup failure (RAM headroom — 6GB required)
- Resolved password login failure (special character issue)
- Confirmed dashboard accessible and operational

### Wazuh Agent (ARM64 → Local Manager)
- Purged corrupted agent installation
- Resolved GPG key import and dpkg pre-removal script failures
- Reinstalled and enrolled against local wazuh-server (192.168.64.5)
- Live telemetry confirmed in dashboard

### Suricata IDS (homeserver)
- Installed Suricata 7.0.3 natively on ARM64
- Configured to monitor enp0s1
- Updated Emerging Threats Open ruleset (49,038 rules)
- Integrated with Wazuh agent via eve.json log collection
- Network-layer detection operational

---

## Key Lessons Learned

- The Wazuh official installer hard-requires x86_64 — ARM64 is rejected
  regardless of available RAM or flags used
- Docker-based Wazuh is unreliable on ARM64 due to image architecture assumptions
- UTM emulation (x86_64 on Apple Silicon) is viable for a SIEM lab workload
- `curl -sO` can fail silently — use `curl -o filename` for reliability
- Wazuh indexer (OpenSearch) needs true 4GB RAM; UTM emulation overhead
  means you must configure 6GB to get sufficient available memory
- Auto-generated passwords with special characters cause browser login failures;
  reset immediately using wazuh-passwords-tool.sh
- Azure credits are a finite resource — local alternatives should be exhausted
  before using cloud infrastructure

---

## Next Steps

- Kali Linux attack simulations against Juice Shop
- Dual-layer detection validation (Suricata + Wazuh endpoint)
- Custom Wazuh rules targeting detection gaps

## Repository Structure

```
00-notes/          Architecture decisions and troubleshooting notes
01-environment-setup/  VM provisioning and OS configuration
02-docker-basics/  Docker installation and container services
03-vulnerable-services/ Juice Shop and attack surface setup
04-wazuh-local/    Local Wazuh SIEM installation and agent enrollment
docs/              Detection validation writeups
```

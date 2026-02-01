# Homelab Security Lab (SOC-Focused)

This repository documents my personal cybersecurity homelab designed to simulate a real-world Security Operations Centre (SOC) environment.

The goal of this lab is to build practical experience in:
- Detection engineering
- Endpoint telemetry and agent management
- Cloud-hosted SIEM deployment
- Incident visibility and troubleshooting

Rather than a single-machine demo, this lab uses a **hybrid architecture** combining a local attack environment with a cloud-hosted SIEM.

---

## High-Level Architecture
```
[Local ARM64 Lab - Apple Silicon]
├─ Ubuntu Server (ARM64)
│ ├─ Docker & Docker Compose
│ ├─ OWASP Juice Shop
│ ├─ Pi-hole
│ └─ Wazuh Agent (4.7.5)
│
├─ Kali Linux (ARM64)
│ └─ Attack simulation
│
└─ Telemetry (TCP 1514)  
            ↓
[Azure VM - x86_64]
├─ Wazuh Manager
├─ Wazuh Indexer
└─ Wazuh Dashboard
```
---

## Environment

### Local Lab
- Host: macOS (Apple Silicon / ARM64)
- Virtualisation: UTM
- Guest OS: Ubuntu Server 22.04 (ARM64)
- Tools:
  - Docker
  - Docker Compose
  - OWASP Juice Shop
  - Pi-hole
  - Kali Linux

### Cloud SIEM
- Platform: Microsoft Azure
- VM Architecture: x86_64
- OS: Ubuntu Server
- SIEM: Wazuh (official installer)

---

## Completed Work

### Docker & Networking
- Installed Docker and Docker Compose on ARM64 Ubuntu
- Verified networking, DNS resolution, and container communication
- Deployed test containers (hello-world, Juice Shop)

### Pi-hole Deployment
- Identified and resolved port conflicts with `systemd-resolved`
- Successfully deployed Pi-hole using Docker
- Validated DNS sinkhole functionality

### Wazuh SIEM Deployment (Azure)
- Provisioned Azure VM for SIEM hosting
- Abandoned Docker-based Wazuh due to certificate and repo instability
- Installed Wazuh Manager, Indexer, and Dashboard using the **official installer**
- Configured TLS and verified dashboard access

### Wazuh Agent Onboarding (ARM → Azure)
- Installed Wazuh agent on ARM64 Ubuntu via APT repository
- Resolved multiple real-world issues:
  - ARM vs AMD64 incompatibilities
  - Rolling repository version skew (agent > manager)
  - Explicit version pinning to Wazuh 4.7.5
  - systemd service corruption after reinstall
  - Missing manager configuration (`MANAGER_IP`)
  - Stale agent keys causing “never connected” state
  - Azure firewall separation of enrollment vs data channels

- Successfully enrolled agent and established live telemetry

---

## Wazuh Agent Onboarding (Key Steps)

```bash
# Install agent (pinned version)
sudo apt install wazuh-agent=4.7.5-1
sudo apt-mark hold wazuh-agent

# Configure manager IP
sudo nano /var/ossec/etc/ossec.conf

# Clean stale keys
sudo rm /var/ossec/etc/client.keys

# Enrol agent
sudo /var/ossec/bin/agent-auth -m <AZURE_IP> -p 1515 -A homeserver

# Restart agent
sudo systemctl restart wazuh-agent
```

Azure inbound rules required:

TCP 1515 – agent enrollment

TCP 1514 – agent telemetry


## Key Lessons Learned

- Docker is not always suitable for complex security platforms
- ARM64 support often requires different installation paths
- Vendor-supported install methods matter
- Rolling repositories require explicit version control
- Agent enrollment and telemetry use different ports
- Logs are more reliable than dashboards when troubleshooting

## Next Steps

- Generate test alerts (SSH failures, sudo abuse, file integrity)
- Validate detections in Wazuh
- Expand to additional agents
- Explore IDS integration (Suricata)

## Why This Lab Exists

This project is intended to demonstrate practical SOC-relevant skills:
- Linux administration
- Cloud security fundamentals
- SIEM deployment and troubleshooting
- Endpoint telemetry and agent lifecycle management
- Detection pipeline thinking

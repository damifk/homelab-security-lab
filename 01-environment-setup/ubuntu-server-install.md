# Ubuntu Server Installation on UTM (homeserver)

## Overview
Ubuntu Server 24.04 ARM64 was deployed as the primary SOC services host
using UTM on macOS Apple Silicon.

## Key Configuration
- Virtualisation: UTM (Apple Hypervisor)
- Architecture: ARM64
- CPU: 2 cores
- Memory: 3 GB
- Storage: 30 GB
- Networking: Shared network with DHCP
- IP Address: 192.168.64.2 (DHCP)

## Role
This host acts as the agent and services platform. It runs:
- Wazuh Agent (4.7.5) — telemetry to local wazuh-server
- OWASP Juice Shop — vulnerable web app for attack simulation
- Pi-hole — DNS sinkhole

It does not run the Wazuh manager/indexer/dashboard — those require x86_64.
See `01-environment-setup/wazuh-server-vm-setup.md` for the dedicated SIEM VM.

## Challenges Encountered
During installation the default Ubuntu mirror intermittently failed to respond.
This was resolved by continuing installation and performing package updates
post-install.

## Outcome
The system successfully obtained a DHCP address and internet connectivity
was verified. Services are deployed via Docker Compose.

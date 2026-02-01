# homelab-security-lab

# Cybersecurity Homelab (SOC & Blue Team)

This repository documents my personal cybersecurity homelab built on a MacBook Air (M4, 16 GB RAM) using virtualisation and containerisation.

The goal of this lab is to gain hands-on experience with security monitoring, attack detection, and incident analysis in a SOC-style environment.

## Lab Environment

- Host: macOS (Apple Silicon)
- Virtualisation: UTM
- Guest OS: Ubuntu Server 22.04
- Container Platform: Docker & Docker Compose
- Attacker VM: Kali Linux (ARM64)

## What This Lab Covers

- Building and managing a lightweight SOC lab
- Deploying containerised security services
- Simulating real-world attacks
- Analysing logs and alerts
- Documenting findings

## Current Progress

- Installed and configured Ubuntu Server on UTM
- Installed Docker Engine and Docker Compose
- Deployed first containerised service (Pi-hole)
- Verified networking and container connectivity
- Pi-hole deployed via Docker Compose  
- Resolved port 53 conflicts with systemd-resolved  
- Exposed Pi-hole dashboard and confirmed healthy container status  
- Deployed Wazuh SIEM on Azure using official installer
- Onboarded ARM64 Ubuntu agent from local lab

## Planned Additions

- IDS integration (Suricata)
- Attack simulations from Kali Linux
- Alert investigation and reporting


## Why This Lab Exists

This homelab demonstrates practical experience with security tooling, Linux administration, Docker, and incident detection beyond theory and certifications.

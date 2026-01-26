# homelab-security-lab

# Cybersecurity Homelab (SOC & Blue Team)

This repository documents my personal cybersecurity homelab built on a MacBook Air (M4, 16 GB RAM) using virtualisation and containerisation.

The goal of this lab is to gain hands-on experience with security monitoring, attack detection, and incident analysis in a SOC-style environment.

## Lab Environment

- Host: macOS (Apple Silicon)
- Virtualisation: UTM
- Guest OS: Ubuntu Server 22.04 (ARM64)
- Container Platform: Docker & Docker Compose
- Attacker VM: Kali Linux (ARM64)

## What This Lab Covers

- Building and managing a lightweight SOC lab on ARM hardware
- Deploying containerised security services
- Simulating real-world attacks
- Analysing logs and alerts
- Documenting findings like a SOC analyst

## Current Progress

- Installed and configured Ubuntu Server on UTM
- Installed Docker Engine and Docker Compose
- Deployed first containerised service (Pi-hole)
- Verified networking and container connectivity

## Planned Additions

- Wazuh SIEM deployment
- IDS integration (Suricata)
- Attack simulations from Kali Linux
- Alert investigation and reporting

## Why This Lab Exists

I am actively pursuing SOC Analyst and Blue Team roles.  
This homelab demonstrates practical experience with security tooling, Linux administration, Docker, and incident detection beyond theory and certifications.

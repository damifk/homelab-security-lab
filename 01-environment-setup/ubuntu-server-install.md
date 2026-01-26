# Ubuntu Server Installation on UTM

## Overview
Ubuntu Server 22.04 ARM64 was deployed as the primary SOC host using UTM on macOS Apple Silicon.

## Key Configuration
- Virtualisation: UTM (Apple Hypervisor)
- CPU: 2 cores
- Memory: 3 GB
- Storage: 30 GB
- Networking: Shared network with DHCP

## Challenges Encountered
During installation the default Ubuntu mirror intermittently failed to respond.  
This was resolved by continuing installation and performing package updates post-install.

## Outcome
The system successfully obtained a DHCP address and internet connectivity was verified.
This host now acts as the central logging and security services platform.

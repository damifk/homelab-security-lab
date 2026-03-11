# Wazuh Server VM Setup (x86_64 via UTM Emulation)

## Overview
A dedicated x86_64 Ubuntu Server VM was created using UTM's emulation mode
on Apple Silicon (ARM64). This VM hosts the full Wazuh stack: manager,
indexer, and dashboard.

## Why a Separate VM
The Wazuh official installer hard-requires x86_64. Both Docker-based and
native installer methods were confirmed incompatible with ARM64. See
`00-notes/architecture-decision-record.md` for full details.

## VM Configuration

| Setting | Value |
|---|---|
| UTM Mode | Emulate (not Virtualise) |
| Architecture | x86_64 |
| System | q35 |
| CPU Cores | 2 |
| RAM | 6144 MB (6GB) |
| Disk | 40 GB |
| Network | Shared Network (DHCP) |
| OS | Ubuntu Server 22.04 LTS (amd64) |
| Hostname | wazuh-server |

## RAM Note
The Wazuh installer requires a minimum of 4GB RAM. With UTM emulation
overhead, a VM configured at 4GB only presents ~3.8GB to the OS, which
causes the wazuh-indexer (OpenSearch) to fail to start. 6GB was required
to provide sufficient headroom.

## Installation

ISO used:
```
ubuntu-22.04.5-live-server-amd64.iso
```

Ubuntu install options selected:
- OpenSSH server enabled during install
- No additional snaps

## Validation

```bash
uname -m       # x86_64
free -h        # ~5.8Gi total
ip a           # 192.168.64.5 (DHCP, may change)
```

## Outcome
VM is running and reachable on the local UTM shared network.
Wazuh installation performed after VM provisioning — see
`04-wazuh-local/wazuh-installation.md`.

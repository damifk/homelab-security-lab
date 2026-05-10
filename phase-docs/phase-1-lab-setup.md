# Phase 1 — Lab Setup

## Objective
Configure VMware Workstation Pro and provision four VMs on an isolated lab network.

## Environment
- **Host:** Windows PC (x86_64, 16GB RAM)
- **Hypervisor:** VMware Workstation Pro (free, post-Broadcom acquisition)
- **Network:** VMnet2, NAT, `192.168.100.0/24`

## Network Configuration
In VMware Virtual Network Editor, VMnet2 was created with the following settings:
- Type: NAT
- Subnet: `192.168.100.0`, Mask: `255.255.255.0`
- Gateway: `192.168.100.2`
- DHCP: enabled
- Host virtual adapter: enabled (required for dashboard access from Windows host)

## VMs Provisioned

| VM | OS | vCPU | RAM | Disk | IP |
|---|---|---|---|---|---|
| wazuh-server | Ubuntu Server 22.04 LTS | 2 (2 cores) | 4GB | 50GB | 192.168.100.128 |
| ubuntu-sensor | Ubuntu Server 24.04 LTS | 2 (1 core) | 2GB | 30GB | 192.168.100.129 |
| kali-attacker | Kali Linux | 2 (1 core) | 2GB | 30GB | 192.168.100.130 |
| windows-endpoint | Windows 10 Pro | 2 (1 core) | 4GB | 60GB | 192.168.100.131 |

All VMs configured with Custom: VMnet2 network adapter. VMware Tools installed on windows-endpoint for display scaling and clipboard integration.

## Notes
- Windows 10 installed without Microsoft account (selected "I don't have internet" during OOBE to avoid forced account creation)
- All privacy settings disabled on Windows 10
- SSH enabled on both Ubuntu VMs during installation

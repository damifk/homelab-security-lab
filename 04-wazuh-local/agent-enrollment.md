# Wazuh Agent Enrollment (ARM64 → Local Manager)

## Overview
The Wazuh agent on the ARM64 homeserver VM was re-enrolled against the local
x86_64 Wazuh manager, replacing the previous Azure-hosted manager.

## Environment
- **Agent Host:** homeserver (Ubuntu 24.04 ARM64)
- **Manager Host:** wazuh-server (Ubuntu 22.04 x86_64, 192.168.64.5)
- **Wazuh Version:** 4.7.5 (both agent and manager)
- **Transport:** TCP 1514 (telemetry), TCP 1515 (enrollment)

## Challenges Encountered

### Broken agent installation (/var/ossec missing)
The previously installed agent had a corrupted state — the package was
registered with dpkg but `/var/ossec` did not exist. This caused all
config and key operations to fail.

### Pre-removal script failure (exit status 127)
The wazuh-agent package could not be removed normally due to a broken
pre-removal script. Resolution:

```bash
sudo rm -f /var/lib/dpkg/info/wazuh-agent.prerm
sudo rm -f /var/lib/dpkg/info/wazuh-agent.postrm
sudo dpkg --remove --force-all wazuh-agent
```

### GPG key import failure
The pipe-based import method failed silently. Resolution — import directly:

```bash
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo gpg --dearmor -o /usr/share/keyrings/wazuh.gpg
sudo chmod 644 /usr/share/keyrings/wazuh.gpg
```

### Ubuntu 24.04 (Noble) — resolv.conf warnings
The homeserver VM was running Ubuntu 24.04, not 22.04 as originally
documented. The agent installer produced `cat: /etc/resolv.conf: No such
file or directory` warnings. These are harmless — Ubuntu 24.04 uses
systemd-resolved without a traditional resolv.conf by default.

## Installation Steps

```bash
# Add Wazuh repository
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo gpg --dearmor -o /usr/share/keyrings/wazuh.gpg
sudo chmod 644 /usr/share/keyrings/wazuh.gpg
echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | sudo tee /etc/apt/sources.list.d/wazuh.list

# Install pinned version
sudo apt update
sudo apt install wazuh-agent=4.7.5-1 -y
sudo apt-mark hold wazuh-agent

# Set manager IP
sudo sed -i 's/<address>MANAGER_IP<\/address>/<address>192.168.64.5<\/address>/' /var/ossec/etc/ossec.conf

# Enroll agent
sudo /var/ossec/bin/agent-auth -m 192.168.64.5 -p 1515 -A homeserver

# Start agent
sudo systemctl daemon-reload
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
```

## Validation

```bash
sudo systemctl status wazuh-agent
# Active: active (running)
# INFO: Valid key received
```

Agent visible in Wazuh dashboard at `https://192.168.64.5:443` as active.

## Outcome
Live telemetry flowing from ARM64 homeserver to local x86_64 Wazuh manager.
No cloud dependency. Full local SOC stack operational.

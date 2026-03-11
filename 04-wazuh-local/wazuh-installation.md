# Wazuh Local Installation (x86_64)

## Overview
Wazuh 4.7.5 was installed on the dedicated x86_64 UTM VM using the official
all-in-one installer. This replaces the previous Azure-hosted SIEM deployment.

## Environment
- **Host VM:** wazuh-server (x86_64, Ubuntu 22.04)
- **IP Address:** 192.168.64.5 (UTM shared network DHCP)
- **Wazuh Version:** 4.7.5
- **Components:** Wazuh Manager, Wazuh Indexer (OpenSearch), Wazuh Dashboard, Filebeat

## Installation Steps

### 1. Download installer and config

```bash
curl -o wazuh-install.sh https://packages.wazuh.com/4.7/wazuh-install.sh
curl -o config.yml https://packages.wazuh.com/4.7/config.yml
```

Note: use `-o` (not `-sO`) to ensure the file is written to disk.

### 2. Configure single-node deployment

```bash
nano config.yml
```

```yaml
nodes:
  indexer:
    - name: node-1
      ip: 127.0.0.1
  server:
    - name: wazuh-1
      ip: 127.0.0.1
  dashboard:
    - name: dashboard
      ip: 127.0.0.1
```

### 3. Run the installer

```bash
sudo bash wazuh-install.sh -a -i
```

The `-i` flag bypasses the hardware recommendation warning (required when
running in an emulated VM that presents slightly less than 4GB).

Installation takes approximately 30-40 minutes in emulated x86_64.

### 4. Reset admin password

The generated password contained special characters that caused login failures
in the browser. Reset to a known password immediately after install:

```bash
sudo /usr/share/wazuh-indexer/plugins/opensearch-security/tools/wazuh-passwords-tool.sh \
  -u admin -p 'YourPasswordHere'
```

Then restart all services:

```bash
sudo systemctl restart wazuh-manager wazuh-indexer wazuh-dashboard filebeat
```

## Challenges Encountered

### curl -sO printed script to screen instead of saving
`-sO` relies on the remote filename for the local filename. In some
environments this fails silently. Fix: use `-o filename` explicitly.

### Installer rejected ARM64 (documented here for reference)
On the ARM64 homeserver VM, the installer exited with:
```
ERROR: Uncompatible system. This script must be run on a 64-bit system.
```
This is a hard x86_64 check, not a hardware spec check. `-i` does not bypass it.

### Indexer failed to start at 4GB RAM
With 4GB configured in UTM, emulation overhead left ~3.8GB available.
OpenSearch requires the full 4GB minimum. Resolution: increase VM RAM to 6GB.

### Generated password login failure
The auto-generated password contained a `?` character which caused
authentication failures in the browser. Resolution: use the
`wazuh-passwords-tool.sh` script to set a clean password post-install.

## Validation

All services running:
```bash
sudo systemctl status wazuh-manager wazuh-indexer wazuh-dashboard filebeat
```

Dashboard accessible at:
```
https://192.168.64.5:443
```

- Accept the self-signed certificate warning
- Login with admin credentials
- Confirm dashboard loads with 0 agents (expected at this stage)

## Outcome
Wazuh SIEM is fully operational locally. No cloud dependency. Azure credits
preserved for dedicated Sentinel/Defender practice.

Next step: re-enroll the ARM64 homeserver agent against this local manager.
See `04-wazuh-local/agent-enrollment.md`.

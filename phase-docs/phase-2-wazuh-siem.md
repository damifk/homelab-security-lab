# Phase 2 — Wazuh SIEM Deployment

## Objective
Deploy Wazuh 4.7.5 all-in-one stack on `wazuh-server`.

## Installation

On `wazuh-server` (Ubuntu Server 22.04, `192.168.100.128`):

```bash
curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh && sudo bash wazuh-install.sh -a
```

The all-in-one installer deploys:
- Wazuh Manager
- Wazuh Indexer (OpenSearch)
- Wazuh Dashboard

Installation takes approximately 10-15 minutes on native x86_64. No flags required (contrast with ARM64 builds which require `-i`).

## Post-Install

Admin credentials printed at end of installation. Password reset to remove special characters that can cause authentication issues:

```bash
sudo /usr/share/wazuh-indexer/plugins/opensearch-security/tools/wazuh-passwords-tool.sh -u admin -p SecureAdmin123
```

Dashboard accessible at `https://192.168.100.128` from Windows host browser.

## Services

All three services confirmed active:
```bash
sudo systemctl status wazuh-manager wazuh-indexer wazuh-dashboard
```

## Notes
- 4GB RAM is sufficient on native x86_64 — the Mac/UTM build required 6GB due to emulation overhead
- OpenSearch binds to localhost only (`127.0.0.1:9200`, `127.0.0.1:9300`) — this is expected
- Dashboard listens on `0.0.0.0:443`
- VMnet2 host virtual adapter must be enabled for the Windows host to reach `192.168.100.128`

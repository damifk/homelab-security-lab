# Architecture Decisions

## Docker vs Native SIEM Deployment

Initial attempts to deploy Wazuh using Docker on ARM64 revealed instability due to:
- Certificate generation mismatches
- Repository structure changes
- Architecture assumptions in container images

Decision:
- Remove Docker-based Wazuh stack from ARM64 host
- Deploy Wazuh using the official installer on a dedicated x86_64 VM

Outcome:
- Stable, supported deployment
- Production-like behaviour
- Full dashboard, indexer, and manager stack running locally

---

## ARM64 Wazuh Manager — Ruled Out

Both deployment methods were attempted on the ARM64 Ubuntu Server VM:

### Docker-based Wazuh (ARM64)
All three containers (wazuh-manager, wazuh-indexer, wazuh-dashboard) entered
a crash-restart loop immediately after creation. Root cause: architecture
assumptions in the official Docker images.

### Official Installer on ARM64
The Wazuh 4.7.5 official install script (`wazuh-install.sh`) performs an
explicit architecture check and hard-exits on non-x86_64 systems:

```
ERROR: Uncompatible system. This script must be run on a 64-bit system.
```

Note: ARM64 is 64-bit, but the installer's check specifically requires x86_64.
The `-i` flag bypasses hardware checks but not the architecture check.

Decision:
- Wazuh manager/indexer/dashboard cannot run on ARM64 via any supported method
- A dedicated x86_64 VM is required for the SIEM stack

---

## x86_64 SIEM VM — Local (UTM Emulation)

Rather than using Azure (as in the previous iteration), a local x86_64 VM was
created using UTM's emulation mode on Apple Silicon.

Trade-offs accepted:
- Emulated x86_64 is slower than native virtualisation
- Acceptable for a lab SIEM with low event volume

Benefits:
- No cloud dependency or credit consumption
- Fully local, air-gappable lab environment
- Azure credits preserved for Sentinel/Defender work

---

## ARM64 Agent Strategy

ARM64 agent installation on the homeserver VM requires:
- APT repository usage instead of direct `.deb`
- Explicit version pinning to match manager version (4.7.5)
- Manual configuration of manager address in `ossec.conf`
- Clean removal of stale `client.keys` before re-enrollment

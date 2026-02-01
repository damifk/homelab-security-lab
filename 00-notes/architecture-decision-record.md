
# Architecture Decisions

## Docker vs Native SIEM Deployment

Initial attempts to deploy Wazuh using Docker on ARM64 revealed instability due to:
- Certificate generation mismatches
- Repository structure changes
- Architecture assumptions

Decision:
- Move SIEM to Azure x86_64
- Use official installer

Outcome:
- Stable, supported deployment
- Production-like behaviour

---

## ARM64 Agent Strategy

ARM64 agent installation required:
- APT repository usage instead of direct `.deb`
- Explicit version pinning to avoid manager mismatch
- Manual configuration of manager address


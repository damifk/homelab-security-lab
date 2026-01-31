# Architecture Decision Record – Wazuh Deployment

## Context
Initial attempts were made to deploy Wazuh using Docker on Apple Silicon (ARM64) and later on an Azure VM.

## Issue
The Docker-based Wazuh single-node stack presented multiple challenges:
- ARM vs amd64 compatibility issues
- Certificate generation inconsistencies
- Docker volume and port conflicts
- Repository changes impacting reproducibility

## Decision
The Wazuh deployment was migrated to an Azure x86_64 virtual machine and installed using the official Wazuh installation script.

## Outcome
- Fully functional Wazuh Manager, Indexer, and Dashboard
- TLS-secured web interface
- Stable and supported deployment
- Clear separation between attack simulation (local lab) and detection (cloud SIEM)

## Rationale
This approach reflects real-world SOC environments where stability, supportability, and clarity outweigh containerisation convenience.

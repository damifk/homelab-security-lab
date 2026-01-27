# Architecture Decision Record – Wazuh Deployment

## Context
Attempted to deploy Wazuh single-node (manager, indexer, dashboard) using Docker on Apple Silicon (ARM64).

## Issues
Persistent container restart loops due to:
- ARM vs amd64 image compatibility
- TLS/bootstrap complexity in Wazuh Docker stack
- Lack of stable, documented ARM support

## Decision
Pivot Wazuh deployment to an Azure x86_64 virtual machine to ensure stability and allow full SOC detection capabilities.

## Outcome
Local lab retained for attack simulation.
Detection stack moved to cloud-based infrastructure.

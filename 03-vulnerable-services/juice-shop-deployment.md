# OWASP Juice Shop Deployment

## Overview
OWASP Juice Shop was deployed as a deliberately vulnerable web application to act as an attack surface for SOC-style testing and detection.

## Why Juice Shop
The application was selected due to:
- Active maintenance
- ARM64 compatibility on Apple Silicon
- Realistic modern web attack patterns

## Deployment
- Deployed using Docker Compose
- Exposed externally on port 8081
- Verified accessible from external hosts on the lab network

## Outcome
The application is stable and reachable, providing a controlled environment for authentication abuse and web attack simulations.

# openvpn-cicd-pipeline
Secure CI/CD pipeline for deploying to private networks via OpenVPN connections
# Secure VPN Deploy

Secure CI/CD pipeline for deploying to private networks via OpenVPN connections

## Overview

This repository provides a GitHub Actions workflow for securely deploying applications to servers behind a VPN. It establishes an OpenVPN connection before deployment, allowing access to private 
networks while maintaining security.

## Setup Instructions

### Prerequisites
- OpenVPN configuration file
- VPN credentials (username and password)
- SSH access to the target server
- GitHub repository with admin access

### GitHub Secrets Setup
Add the following secrets to your GitHub repository:
- `OVPN_CONFIG_B64`: Base64-encoded OpenVPN config file
- `OVPN_USERNAME`: VPN username
- `OVPN_PASSWORD`: VPN password
- `HOST_DNS`: Internal DNS/IP of deployment server
- `USERNAME`: SSH username
- `SSH_PRIVATE_KEY`: SSH private key for authentication

### How to Use
1. Fork/clone this repository
2. Configure your GitHub secrets
3. Customize the deployment script in `.github/workflows/deploy.yml`
4. Push changes to trigger deployment

## Security Considerations
- Use dedicated VPN and SSH credentials for CI/CD
- Regularly rotate credentials
- Restrict VPN access to only necessary resources

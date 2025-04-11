# Setting Up OpenVPN in GitHub CI/CD Pipelines: A Complete Guide

In modern DevOps environments, secure deployment to private networks is often a requirement. This guide explains how to integrate OpenVPN connections into your GitHub CI/CD pipeline to securely deploy applications to systems behind a VPN.

## Why Use OpenVPN in CI/CD?

Using OpenVPN within your CI/CD workflow provides several benefits:

1. **Security**: Deploy to internal networks without exposing them to the public internet
2. **Access Control**: Maintain strict access policies while allowing automated deployments
3. **Consistency**: Deploy to the same environment for testing and production
4. **Simplicity**: Avoid complex network routing or additional gateway servers

## Prerequisites

- An OpenVPN configuration file (`.ovpn`)
- VPN credentials (username and password)
- A GitHub repository where you'll set up the CI/CD workflow
- Admin access to configure GitHub repository secrets
- SSH credentials for the target deployment server

## Setting Up Your Local Environment

Before setting up the GitHub Actions workflow, let's prepare and test the OpenVPN configuration locally.

### Step 1: Test Your OpenVPN Connection

First, verify that your OpenVPN configuration works:

> sudo openvpn --config vpn-config-file.ovpn

### Step 2: Store Username and Password

 Test Authentication with Credentials File

> nano auth.txt
> Enter username on line 1, password on line 2

### Step 3: Test the connection using the authentication file:

> sudo openvpn --config vpn-config-file.ovpn --auth-user-pass auth.txt

### Step 4: Encode the OpenVPN Configuration File

For securely storing the OpenVPN configuration in GitHub Secrets, encode it to base64:

Base64-encode the config file for GitHub Secrets:

> base64 vpn-config-file-new.ovpn > ovpn.b64
> cat ovpn.b64

### Setting Up GitHub Repository Secrets

GitHub Secrets provide a secure way to store sensitive information like VPN credentials. Go to your GitHub repository, then:

Navigate to Settings > Secrets and variables > Actions : Click New repository secret

    Add the following secrets:

| Secret Name         | Value                                        |
| ------------------- | -------------------------------------------- |
| `OVPN_CONFIG_B64` | Content of your `ovpn.b64`file             |
| `OVPN_USERNAME`   | Your VPN username                            |
| `OVPN_PASSWORD`   | Your VPN password                            |
| `HOST_DNS`        | The internal DNS/IP to verify VPN connection |
| `USERNAME`        | SSH username for deployment server           |
| `SSH_PRIVATE_KEY` | Your SSH private key for authentication      |

### VPN Hostname

GitHub Secret that holds the hostname or IP address of a server inside your VPN —basically, a way to confirm that the VPN connection was successful.`

* A **private IP** behind your VPN (e.g., `10.0.0.1`)
* A **hostname** of an internal resource (e.g., `internal-server.local`)

### Create a workflow file in .github/workflows/deploy.yml:

```
name: Build and Deploy via VPN

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
  
      - name: Build application
        run: |
          # Your build steps here
          echo "Building application..."
          # Example: npm install && npm run build
  
  deploy:
    needs: build
    runs-on: ubuntu-latest
    name: Deploy Application via VPN & SSH
    steps:
      - name: Checkout Code
        uses: actions/checkout@v4
  
      - name: Install & Connect to OpenVPN
        run: |
          sudo apt update
          sudo apt install -y openvpn 
          echo "${{ secrets.OVPN_CONFIG_B64 }}" | base64 -d > sslvpn.ovpn
          echo "${{ secrets.OVPN_USERNAME }}" > auth.txt
          echo "${{ secrets.OVPN_PASSWORD }}" >> auth.txt
          sudo openvpn --config sslvpn.ovpn --auth-user-pass auth.txt --daemon
          sleep 15  # wait for VPN connection to establish
  
      - name: Verify VPN Connection
        run: |
          curl -v ${{ secrets.HOST_DNS }} || (echo "VPN connection failed!" && exit 1)
  
      - name: Deploy Application to VPS via SSH
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.HOST_DNS }}
          username: ${{ secrets.USERNAME }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            echo "Deploying application..........."
            # Add your deployment commands here
            # Examples:
            # cd /path/to/app
            # git pull
            # docker-compose up -d
            # systemctl restart your-service
```

### How the Workflow Works:

Let's break down the important steps in the workflow:

1. **Build Job** : Compiles your application (customize this for your specific project)
2. **Deploy Job** : Runs after the build completes successfully

* Installs OpenVPN on the runner
* Decodes the base64 OpenVPN configuration
* Creates an authentication file with the VPN credentials
* Starts OpenVPN in daemon mode
* Waits for the connection to establish
* Verifies the VPN connection by attempting to reach an internal resource
* Connects to the private server via SSH using appleboy/ssh-action
* Executes deployment commands on the remote server

### Connecting to Private SSH After VPN Establishment:

The key addition to our workflow is the SSH connection after the VPN is established. The `appleboy/ssh-action` GitHub Action provides a convenient way to execute commands on a remote server over SSH.

Important points about the SSH deployment step:

1. **Host** : We use the same `HOST_DNS` secret that we used to verify the VPN connection
2. **Authentication** : We use SSH key-based authentication for better security
3. **Username:** Contains server username(root) connect to the remote server

### Conclusion:

Integrating OpenVPN into your GitHub CI/CD pipeline enables secure deployments to internal networks while maintaining your security posture. By adding SSH deployment after establishing the VPN connection, you create a fully automated and secure deployment pipeline to private infrastructure. This approach gives you the benefits of modern CI/CD practices even when working with private or restricted networks, without compromising on security or automation.


Remember to customize the build and deployment steps according to your specific application requirements, and always follow security best practices when handling VPN and SSH credentials.

 **Happy deploying!**

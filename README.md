# AWS VPN Deployment Pipeline

This repository contains an automated setup using GitHub Actions to deploy an OpenVPN server on an AWS EC2 instance. After the server is deployed, it automatically creates a Route53 DNS entry in a secondary AWS account. 

The VPN setup is powered by Docker and the [kylemanna/openvpn](https://github.com/kylemanna/docker-openvpn) image.

## Features

1. **Automated EC2 VPN Deployment**: Spins up an Amazon Linux 2023 instance with Docker and starts an OpenVPN server via AWS CloudFormation.
2. **Cross-Account DNS Update**: A secondary job configures Route 53 in a different AWS account.
3. **Approval Gate for DNS**: The DNS update is protected by a GitHub Environment approval step. 

## Prerequisites & Setup Instructions

Before running the workflow, you must set up your repository settings properly.

### 1. Create a GitHub Environment

To implement the manual approval for the DNS update step:

1. Go to your repository **Settings** > **Environments**.
2. Click **New environment** and name it `dns-approval`.
3. Check the **Required reviewers** box and add yourself (or appropriate team members).
4. Save the protection rules.

### 2. Configure GitHub Secrets

You must add the following **Repository Secrets** (in **Settings** > **Secrets and variables** > **Actions**):

#### VPN Account (VPN Hosting)
- `AWS_ACCESS_KEY_ID_VPN_ACCOUNT`: AWS Access Key for the account where the EC2 instance goes.
- `AWS_SECRET_ACCESS_KEY_VPN_ACCOUNT`: AWS Secret Key for VPN Account.
- `AWS_REGION_VPN_ACCOUNT`: Region to deploy the VPN (e.g. `us-east-1`).

#### Main Account (DNS Hosting)
- `AWS_ACCESS_KEY_ID_MAIN_ACCOUNT`: AWS Access Key for the account where Route 53 is managed.
- `AWS_SECRET_ACCESS_KEY_MAIN_ACCOUNT`: AWS Secret Key for Main Account.
- `AWS_REGION_MAIN_ACCOUNT`: Region where Route 53 configuration happens (e.g. `us-east-1`).
- `DNS_HOSTED_ZONE_ID`: The Route 53 Hosted Zone ID in Main Account (e.g., `Z111111QQQQQQQ`).
- `VPN_DOMAIN_NAME`: The domain name to assign to your VPN IP (e.g., `vpn.mydomain.com`).

#### VPN Configuration
- `VPN_USER`: The username for the VPN. This is used to name your generated `.ovpn` configuration file.
- `VPN_PASSWORD`: A password you choose to encrypt your `.ovpn` configuration file. You will be prompted to enter this password the very first time you import the profile into the OpenVPN Connect app.

#### EC2 Server Access (Optional but Recommended)
- `SSH_PUBLIC_KEY`: Your personal SSH public key (e.g., the contents of `~/.ssh/id_rsa.pub` or `~/.ssh/id_ed25519.pub`). This guarantees you can SSH into the EC2 instance as the `ec2-user` if you ever need to debug the server.

## How to Deploy

1. Ensure all your secrets and the `dns-approval` environment are correctly set up.
2. Go to the **Actions** tab in GitHub.
3. Select **Deploy VPN Server** workflow from the left sidebar.
4. Click **Run workflow** -> Hit Run.
5. After the EC2 deployment succeeds, the workflow will pause for manual approval.
6. Approve the deployment to the `dns-approval` environment so that the DNS update occurs in Main Account.

## How to Destroy

To tear down the VPN infrastructure and clean up the DNS records:
1. Go to the **Actions** tab in GitHub.
2. Select **Destroy VPN Server** workflow from the left sidebar.
3. Click **Run workflow** -> Hit Run.
4. Wait for the workflow to complete. It will delete the DNS records from Main Account and the CloudFormation stack from VPN Account.

## Connecting to the VPN

This setup uses OpenVPN. You do not need to manually configure any IP addresses or shared secrets on your devices!

1. After you click **Deploy VPN Server** and the GitHub Action run completes successfully, scroll to the bottom of the run summary page.
2. Under the **Artifacts** section, you will see a file named `OpenVPN-Profile`. Download and extract this `.zip` file. Inside, you will find your generated `<VPN_USER>.ovpn` file.
3. Download the official, free [OpenVPN Connect Client](https://openvpn.net/client/) for your Mac, Windows, iOS, or Android device.
4. Double-click or import the `.ovpn` file into the OpenVPN Connect app.
5. Click **Connect** and you're secure!

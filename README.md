# 🛡️ Secure SSH Access with Domain via Cloudflare Zero Trust Tunnel

A comprehensive guide and documentation on setting up a secure, encrypted SSH connection with your Domain to a remote **AlmaLinux** server without opening any inbound ports (No Port Forwarding) 
or exposing the server's public IP.

## 🚀 Project Overview
This project demonstrates how to use **Cloudflare Zero Trust (Argo Tunnel)** to bypass ISP restrictions and NAT firewalls. 
It provides a "Zero Trust" security model where the server is completely hidden from the public internet, and only authorized users can access it via a secure tunnel.

## 🛠️ Tech Stack
* **Server OS:** AlmaLinux 9 (RHEL-based)
* **Security Layer:** Cloudflare Zero Trust
* **Tunneling:** `cloudflared` (Argo Tunnel)
* **Client OS:** Windows 11 / Linux
* **Protocol:** SSH (Secure Shell)

---
🏗️ Architecture Overview
Traditional SSH requires opening Port 22 to the internet, which is risky. This project uses a Reverse Tunnel:

The server initiates an outbound connection to Cloudflare.

Cloudflare acts as a proxy.

The client connects to Cloudflare, which pipes the traffic to the server.

## 🛠️ Prerequisites
A domain name (e.g., mridu.me) added to a Cloudflare account.

A server running AlmaLinux 9.

A Windows/Linux laptop for the client side.

## 📡 Phase 1: Cloudflare Dashboard Setup
Log in to the Cloudflare Zero Trust Dashboard.

https://dash.cloudflare.com/c7ff14348b1847c3c4ef3e76db18dbaf/one/networks/connectors

**Navigate to Networks > Tunnels and click Create a Tunnel.

Select Cloudflare Managed (Dashboard).

Name the tunnel: my-alma-server.

Save Tunnel and you will see the installation commands for different OS. Copy the Token from the command.**

## 🐧Phase 2: Server-Side Installation (AlmaLinux)
Login to your AlmaLinux terminal and follow these steps:

## 1. Install cloudflared agent
```bash
# Download the RPM package
curl -L --output cloudflared.rpm https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-x86_64.rpm
```
## Install the package
```bash
sudo yum localinstall -y cloudflared.rpm
```
## 2. Connect to the Tunnel
Paste the token command you copied from Phase 1:

```bash
sudo cloudflared service install <YOUR_SECRET_TOKEN>
```

## 3. Networking Fix (Crucial Step)
If you encounter Network is unreachable, it's likely due to IPv6 issues with your ISP. Disable it:

```bash
sudo sysctl -w net.ipv6.conf.all.disable_ipv6=1
sudo sysctl -w net.ipv6.conf.default.disable_ipv6=1
```
## 4. Verify Service
```bash
sudo systemctl start cloudflared
sudo systemctl status cloudflared
```
## 🔐 Phase 3: Security & Access Policy
Back in the Cloudflare Dashboard:

## 1. Set Public Hostname
Go to your Tunnel Configuration > Public Hostname.

Subdomain: ssh (Optional, or leave blank for root domain mridu.me).

Domain: mridu.me.

Service: SSH.

URL: localhost:22.

## 2. Create Access Application
This ensures only YOU can trigger the tunnel.

Go to Access > Applications > Add an Application.

Select Self-hosted.

Domain: mridu.me.

Policy: Action: Allow, Selector: Emails, Value: your-email@gmail.com.

## 💻 Phase 4: Client-Side Configuration (Windows)
# 1. Install cloudflared on Windows
Download the .msi from Cloudflare GitHub:https://github.com/cloudflare/cloudflared/releases

**Run the installer and open PowerShell as Admin.**

# 2. DNS & Connection Setup
If your ISP blocks Cloudflare domains (no such host error):

Set DNS to 8.8.8.8 (Google).

Flush DNS: ipconfig /flushdns.

Connect VPN (if ISP blocks the initial handshake).

# 3. Authenticate Client
```bash
PowerShell
cloudflared access login https://mridu.me

```

## 4. Final SSH Connection
```bash
PowerShell
ssh -o "ProxyCommand=cloudflared access ssh --hostname mridu.me" mridu@mridu.me
```

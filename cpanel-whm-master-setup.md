# 🚀 cPanel & WHM Master Setup Guide (2026)
Host: host.yourdomain.tld
OS: AlmaLinux 9

### 1️⃣ Cloudflare DNS (The Foundation)
'''Before installing cPanel, configure DNS properly.'''

Go to Cloudflare → DNS and add:
Type	Name	Content	Proxy Status
A	server	YOUR_GCP_IP	🔘 DNS Only (Grey Cloud)
A	whm	YOUR_GCP_IP	🔘 DNS Only (Grey Cloud)

⚠️ Important: Do NOT proxy WHM (2087) through normal orange-cloud unless using Cloudflare Tunnel. Direct proxy will break WHM SSL.

### 2️⃣ OS Preparation (AlmaLinux 9)

Login as root via SSH.

2.1 Convert to AlmaLinux (Skip if Already AlmaLinux)
curl -O https://raw.githubusercontent.com/AlmaLinux/almalinux-deploy/master/almalinux-deploy.sh
bash almalinux-deploy.sh
reboot
2.2 Verify OS
cat /etc/redhat-release

Should display AlmaLinux 9.

2.3 Set Hostname (FQDN Required)
hostnamectl set-hostname HOST.YOURDOMAIN.TLD

Verify:

hostname -f
2.4 SELinux (Correct Approach)

⚠️ cPanel supports permissive mode, full disabling is not required.

Recommended:

setenforce 0
sed -i 's/^SELINUX=.*/SELINUX=permissive/' /etc/selinux/config

Reboot:

reboot

### 3️⃣ cPanel & WHM Installation
3.1 Install Required Packages
dnf update -y
dnf install perl curl screen -y
3.2 Start Screen Session
screen -S cpanel_install
3.3 Run Installer
cd /home
curl -o latest -L https://securedownloads.cpanel.net/latest
sh latest

Installation takes 30–60 minutes.

Access WHM:

https://YOUR_SERVER_IP:2087
4️⃣ Disable BIND (Using Cloudflare DNS Only)

Since you're using Cloudflare DNS, you do NOT need BIND.

In WHM:

WHM → Service Manager

Uncheck:

❌ named (BIND DNS Server)

Then:

WHM → Basic WebHost Manager Setup

Set:

Nameservers → Leave empty or use Cloudflare placeholder

### 5️⃣ Cloudflare Tunnel (Admin Shield 🔐)

This hides WHM from public IP completely.

5.1 Install Cloudflared
curl -fsSl https://pkg.cloudflare.com/cloudflared.repo | tee /etc/yum.repos.d/cloudflared.repo
dnf install cloudflared -y
5.2 Authenticate
cloudflared tunnel login
5.3 Create Tunnel
cloudflared tunnel create cp-tunnel

Copy the <UUID>.

5.4 Create Config File

/root/.cloudflared/config.yml

tunnel: <UUID>
credentials-file: /root/.cloudflared/<UUID>.json

ingress:
  - hostname: whm.YOURDOMAIN.TLD
    service: https://localhost:2087
    originRequest:
      noTLSVerify: true
  - service: http_status:404
5.5 Route DNS
cloudflared tunnel route dns cp-tunnel whm.YOURDOMAIN.TLD
5.6 Install Service
cloudflared service install
systemctl enable cloudflared
systemctl start cloudflared

### 6️⃣ Firewall (External Lockdown)

In VPS Firewall Manager:

Allow only:

TCP: 80, 443
TCP: 465, 587
TCP: 993, 995

Remove:

2087 (WHM)
2083 (cPanel)
22 (SSH) – unless restricted to your IP only

⚠️ If disabling SSH public access, use:

Cloudflare Tunnel SSH

Or restrict SSH to your static IP only

# 7️⃣ Server DDoS Protection
7.1 Network Level

Use GCP Shielded VM

Enable provider DDoS protection

Use Cloudflare proxy for websites

7.2 Application Level

In WHM:

Security Center → cPHulk Brute Force Protection

Enable:

Login protection

IP-based lockouts

Install:

WHM → Plugins → ConfigServer Security & Firewall (CSF)

Enable:

SYN flood protection

Port flood protection

Connection tracking

### 8️⃣ Cloudflare Access (Final Guard 🛡)

Cloudflare Zero Trust → Access → Applications

Add:

whm.YOURDOMAIN.TLD

Policy:

Email OTP

Now WHM login is protected by:

Cloudflare Access

Tunnel

WHM login itself

Triple-layer security.

## 🚀 cPanel & WHM Master Setup Guide (2026)

Host: `subhost.yourdomain.tld`

OS: **AlmaLinux 9**

DNS: **Cloudflare (External DNS – No BIND usage)**

#### 1️⃣ Cloudflare DNS (The Foundation)

_Before installing cPanel, configure DNS properly._

Go to **Cloudflare** → **DNS** and add:
Type Name Content Proxy Status
A server YOUR_GCP_IP 🔘 DNS Only (Grey Cloud)
A whm YOUR_GCP_IP 🔘 DNS Only (Grey Cloud)

⚠️ Important: **Do NOT proxy WHM (2087) through normal orange-cloud unless using Cloudflare Tunnel. Direct proxy will break WHM SSL.**

#### 2️⃣ OS Preparation (AlmaLinux 9)

Login as root via SSH.

2.1 - Convert to AlmaLinux (Skip if Already AlmaLinux)

    curl -O https://raw.githubusercontent.com/AlmaLinux/almalinux-deploy/master/almalinux-deploy.sh
    bash almalinux-deploy.sh
    reboot

2.2 - Verify OS

    cat /etc/redhat-release

Should display AlmaLinux 9.

2.3 - Set Hostname (FQDN Required)

    hostnamectl set-hostname HOST.YOURDOMAIN.TLD

    # Verify:
    hostname -f

2.4 SELinux (Correct Approach)

_⚠️ cPanel supports permissive mode, full disabling is not required._

    setenforce 0
    sed -i 's/^SELINUX=.*/SELINUX=permissive/' /etc/selinux/config
    reboot

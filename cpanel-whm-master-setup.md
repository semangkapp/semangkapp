## 🚀 cPanel & WHM Master Setup Guide (2026)

Host: `subhost.yourdomain.tld` | OS: **AlmaLinux 9** | DNS: **Cloudflare (External DNS – No BIND usage)**

---

### 1️⃣ Cloudflare DNS (The Foundation)

_Before installing cPanel, configure DNS properly._

Go to **Cloudflare** → **DNS** and add:

<table>
<tr>
<th>Type</th><th>Name</th><th>Content</th><th>Proxy</th><th>Status</th>
</tr>
<tr>
<td>A</td><td>server</td><td>YOUR_GCP_IP</td><td>🔘</td><td>DNS Only (Grey Cloud)</td>
</tr>
<tr>
<td>A</td><td>whm</td><td>YOUR_GCP_IP</td><td>🔘</td><td>DNS Only (Grey Cloud)</td>
</tr>
</table>

⚠️ Important: **Do NOT proxy WHM (2087) through normal orange-cloud unless using Cloudflare Tunnel. Direct proxy will break WHM SSL.**

---

### 2️⃣ OS Preparation (AlmaLinux 9)

Login as root via SSH.

#### 2.1 - Convert to AlmaLinux (Skip if Already AlmaLinux)

    curl -O https://raw.githubusercontent.com/AlmaLinux/almalinux-deploy/master/almalinux-deploy.sh
    bash almalinux-deploy.sh
    reboot

#### 2.2 - Verify OS

    cat /etc/redhat-release

_Should display AlmaLinux 9._

#### 2.3 - Set Hostname (FQDN Required)

    hostnamectl set-hostname subhost.yourdomain.tld

    # Verify:
    hostname -f

#### 2.4 - SELinux (Correct Approach)

_⚠️ cPanel supports permissive mode, full disabling is not required._

    setenforce 0
    sed -i 's/^SELINUX=.*/SELINUX=permissive/' /etc/selinux/config
    reboot

---

### 3️⃣ cPanel & WHM Installation

#### 3.1 - Install Required Packages

    dnf update -y
    dnf install perl curl screen git wget fail2ban python3 python-is-python3  -y

#### 3.2 Start Screen Session

    screen -S cpanel_install

#### 3.3 - Run Installer

    cd /home
    curl -o latest -L https://securedownloads.cpanel.net/latest
    sh latest

_Installation takes 30–60 minutes._

Access WHM:

    https://YOUR_SERVER_IP:2087

---

### 4️⃣ Disable BIND (Using Cloudflare DNS Only)

Since you're using Cloudflare DNS, you do NOT need BIND.

In WHM:

    WHM → Service Manager

Uncheck:

- ❌ named (BIND DNS Server)

Then:

    WHM → Basic WebHost Manager Setup

Set:

    Nameservers → Leave empty or use Cloudflare placeholder

---

### 5️⃣ Cloudflare Tunnel (Admin Shield 🔐)

_This hides WHM from public IP completely._

#### 5.1 - Install Cloudflared

    curl -fsSl https://pkg.cloudflare.com/cloudflared.repo | tee /etc/yum.repos.d/cloudflared.repo
    dnf install cloudflared -y

#### 5.2 - Authenticate

    cloudflared tunnel login

#### 5.3 - Create Tunnel

    cloudflared tunnel create cp-tunnel

Copy the <UUID>.

#### 5.4 - Create Config File

`/root/.cloudflared/config.yml`

    tunnel: <UUID>
    credentials-file: /root/.cloudflared/<UUID>.json

    ingress:
      - hostname: whm.yourdomain.tld
        service: https://localhost:2087
        originRequest:
          noTLSVerify: true
      - service: http_status:404

#### 5.5 - Route DNS

    cloudflared tunnel route dns cp-tunnel whm.yourdomain.tld

#### 5.6 - Install Service

    cloudflared service install
    systemctl enable cloudflared
    systemctl start cloudflared

---

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

- Cloudflare Tunnel SSH
- Or restrict SSH to your static IP only

---

### 7️⃣ Server DDoS Protection

#### 7.1 - Network Level

- Use GCP Shielded VM
- Enable provider DDoS protection
- Use Cloudflare proxy for websites

#### 7.2 - Application Level

In WHM:

    Security Center → cPHulk Brute Force Protection

Enable:

- Login protection
- IP-based lockouts

Install:

    WHM → Plugins → ConfigServer Security & Firewall (CSF)

Enable:

- SYN flood protection
- Port flood protection
- Connection tracking

---

### 8️⃣ Cloudflare Access (Final Guard 🛡)

Cloudflare Zero Trust → Access → Applications

Add:

    whm.yourdomain.tld

Policy:

    Email OTP

Now WHM login is protected by:

1. Cloudflare Access
2. Tunnel
3. WHM login itself

Triple-layer security.

---

### 9️⃣ 8GB RAM — Do We Need "Starvation Mode"?

**No.** If your vpn 8GB RAM is comfortable for cPanel.

You do NOT need extreme stripping.

But you should avoid bloat.

Ideal Philosophy:

- Lean but not crippled
- Performance > Features
- Disable what you don't use

---

### 🔟 WHM Performance Optimization (HTML Powerhouse Mode)

Since you:

- ❌ Don't use WordPress
- ❌ Don't use heavy databases
- ❌ Minimal email usage
- ✅ Serve mostly static HTML

We optimize for:

**Maximum Apache/Nginx efficiency**

**Minimal MySQL footprint**

**Low background services**

#### 10.1 - MySQL (If Barely Used)

Edit /etc/my.cnf:

    [mysqld]
    innodb_buffer_pool_size=512M
    max_connections=50
    query_cache_type=0

Restart:

    systemctl restart mysql

#### 10.2 - Apache Optimization

WHM → EasyApache 4

Use:

- MPM Event
- PHP-FPM only (if needed)
- Disable unused PHP versions
- Remove mod_ruid2 if not needed

Apache Config:

    KeepAlive On
    MaxKeepAliveRequests 100
    KeepAliveTimeout 2

#### 10.3 - Disable Unused Services

WHM → Service Manager

Disable if unused:

❌ FTP (if not needed)
❌ WebDisk
❌ CalDAV/CardDAV
❌ SpamAssassin (if no email use)

---

### 1️⃣1️⃣ Resource Surgery (Maximum Performance Mode)

#### 11.1 Swap Setup (If None Exists)

    fallocate -l 2G /swapfile
    chmod 600 /swapfile
    mkswap /swapfile
    swapon /swapfile

Persist:

    echo '/swapfile none swap sw 0 0' >> /etc/fstab

#### 11.2 Tune Kernel

`/etc/sysctl.conf`

    vm.swappiness=10
    net.core.somaxconn=65535
    net.ipv4.tcp_max_syn_backlog=4096

Apply:

    sysctl -p

#### 11.3 Disable Unused PHP Modules

Remove:

- intl (if not needed)
- imap
- ldap
- sqlite (if unused)

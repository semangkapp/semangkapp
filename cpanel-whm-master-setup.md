🚀 cPanel & WHM Master Setup Guide (2026)

Host: host.yourdomain.tld
OS: AlmaLinux 9
DNS: Cloudflare (External DNS – No BIND usage)

1️⃣ Cloudflare DNS (The Foundation)

Before installing cPanel, configure DNS properly.

Go to Cloudflare → DNS and add:

Type	Name	Content	Proxy Status
A	server	YOUR_GCP_IP	🔘 DNS Only (Grey Cloud)
A	whm	YOUR_GCP_IP	🔘 DNS Only (Grey Cloud)

⚠️ Important: Do NOT proxy WHM (2087) through normal orange-cloud unless using Cloudflare Tunnel. Direct proxy will break WHM SSL.

# 🍓 Pi-hole Network-Wide Ad Blocker

> DNS-level ad blocking and network traffic monitoring for all devices on your home network — powered by a Raspberry Pi.

![Pi-hole Dashboard](screenshots/pihole-dashboard-main.png)

---

## 👋 About This Project

I'm a Computer Science student with a passion for exploring technology at every level — from low-level systems and networking to security and infrastructure. I got my hands on a Raspberry Pi and wanted to put it to real use rather than let it collect dust. Pi-hole stood out because it wasn't just a fun experiment; it was something that would directly benefit my entire home network. Setting it up pushed me to get hands-on with DNS, network traffic analysis, Linux administration, and systemd services. What surprised me most was how much background noise exists on a typical home network — 16% of all DNS queries were blocked, and the network noticeably felt faster once all that traffic was cut off at the source. This project gave me practical exposure to the kind of infrastructure-level thinking that matters in cybersecurity, SRE, and Linux-focused roles.

---

## 📊 Live Stats

| Metric | Value |
|---|---|
| Total DNS Queries | 21,688 |
| Queries Blocked | 3,469 |
| Block Rate | 16.0% |
| Domains on Blocklist | 79,079 |
| Active Clients | 7 |

---

## 🔍 What Is Pi-hole?

Pi-hole is a network-level ad blocker that acts as a **DNS sinkhole** — intercepting DNS queries from every device on your network and blocking requests to known ad-serving, tracking, and malicious domains. Unlike browser extensions, Pi-hole works on **all devices and all apps**, including phones, smart TVs, and IoT devices — with zero configuration on each device.

---

## ✨ Features

- **Network-wide blocking** — protects every device without installing anything on them
- **DNS-level filtering** — blocks ads in apps, not just browsers
- **Real-time query log** — see exactly what every device on your network is requesting
- **Web dashboard** — monitor statistics, manage blocklists, and configure settings from a browser
- **Upstream DNS routing** — forwards non-blocked queries to Google (8.8.8.8) and Cloudflare (1.1.1.1)
- **DNSSEC support** — validates DNS responses to prevent spoofing
- **Bypass prevention** — blocks Firefox DoH, Apple iCloud Private Relay, and DDR bypass attempts
- **DHCP server** — Pi-hole manages IP assignments across the entire network
- **Runs 24/7 on low power** — Raspberry Pi uses only ~5W

---

## 🌐 How It Works

![Network Diagram](screenshots/network-diagram.svg)

Every device on the network sends DNS queries to the router. The router forwards all DNS traffic to Pi-hole. Pi-hole checks each domain against its blocklist (81,131 domains) — blocked domains get an immediate `NXDOMAIN` response, so the ad or tracker never loads. Allowed domains are forwarded to upstream resolvers (Google 8.8.8.8 or Cloudflare 1.1.1.1, both with DNSSEC), and the real IP is returned normally. No per-device configuration is needed.

---

## 🛠️ Hardware

| Component | Details |
|---|---|
| Device | Raspberry Pi (hostname: `PakisPI`) |
| Architecture | `linux/arm64/v8` |
| OS | Raspberry Pi OS |
| Storage | MicroSD |
| Power | ~5W continuous |

---

## ⚙️ Configuration

| Setting | Value |
|---|---|
| Pi-hole Core | v6.3 |
| Web Interface | v6.4 |
| FTL Engine | v6.4.1 |
| Upstream DNS | Google (ECS, DNSSEC) + Cloudflare (DNSSEC) |
| Gravity Domains | 81,131 |
| DNS Port | 53 (UDP + TCP, IPv4 + IPv6) |
| Web Port | 80 / 443 |
| DHCP Server | Enabled |
| Privacy Level | 1 (domains hidden in logs) |
| Blocking Status | ✅ Enabled |
| Service Uptime | Active since 2026-03-12 |

See [`config/pihole.toml`](config/pihole.toml) for the full annotated configuration file with all modified settings highlighted.

---

## 🖥️ System Health

![System Health](screenshots/pihole-system-health.png)
*CPU temperature, memory usage, disk, and uptime captured directly from the Pi — confirming the full stack runs comfortably within hardware limits at 47.2°C and 27% memory usage.*

---

## 📂 Repository Structure

```
pihole-network-monitor/
├── README.md                    # This file
├── SETUP.md                     # Full installation & configuration guide
├── .gitignore
├── config/
│   └── pihole.toml              # Annotated config (sensitive values redacted)
└── screenshots/
    ├── pihole-dashboard-main.png
    ├── pihole-login-page.png
    ├── pihole-query-log-blocked.png
    ├── pihole-query-log-internal.png
    ├── pihole-dns-upstream-servers.png
    ├── pihole-listening-ports.png
    ├── pihole-command-line.png
    ├── pihole-system-health.png
    └── network-diagram.svg
```

---

## 🚀 Quick Start

See [SETUP.md](SETUP.md) for the full installation and configuration guide.

**One-line installer:**
```bash
curl -sSL https://install.pi-hole.net | bash
```

---

## 🖥️ Screenshots

### Login Page
![Login Page](screenshots/pihole-login-page.png)

### Query Log — Blocked Requests
Domains flagged by the blocklist appear with a red stop icon. Domain names are hidden in this screenshot for privacy.

![Query Log - Blocked](screenshots/pihole-query-log-blocked.png)

### Query Log — Internal Queries
Queries originating from the Pi-hole itself (shown as `pi.hole`) are resolved locally with sub-millisecond response times.

![Query Log - Internal](screenshots/pihole-query-log-internal.png)

### Upstream DNS Servers
Configured to use **Google (ECS, DNSSEC)** and **Cloudflare (DNSSEC)** as upstream resolvers.

![Upstream DNS](screenshots/pihole-dns-upstream-servers.png)

### Service Status & Listening Ports
Pi-hole FTL listens on port **53** (DNS) over UDP and TCP for both IPv4 and IPv6, port **80** (HTTP), and port **443** (HTTPS).

![Listening Ports](screenshots/pihole-listening-ports.png)

### Command Line Status
Pi-hole running on **Core v6.3**, blocking enabled, with **81,131 gravity domains** loaded.

![Command Line](screenshots/pihole-command-line.png)

---

## 💡 Results & Observations

After running Pi-hole for 24 hours across 7 active clients:

- **16% of all DNS queries were blocked** — higher than expected. Most home users have no visibility into how much ad and tracking traffic their devices generate constantly in the background.
- **Network felt noticeably faster** — eliminating DNS lookups for blocked domains removes a layer of latency that adds up across hundreds of queries per hour.
- **Pi-hole also acts as the DHCP server** — centralizing both DNS and DHCP on one device simplified network management and gave full visibility into every client on the network by hostname.
- **Three bypass methods were explicitly blocked** — Firefox DNS-over-HTTPS, Apple iCloud Private Relay, and Discovery of Designated Resolvers were all configured to prevent devices from routing around Pi-hole without my knowledge.

---

## 🎓 What I Learned

- **DNS and how the internet actually works** — setting up Pi-hole required understanding the full DNS resolution chain, from device query to upstream resolver to returned IP, and where interception is possible at each step.
- **Linux administration** — managing a 24/7 service via systemd, checking logs, configuring network interfaces, and using the command line as the primary interface rather than a GUI.
- **Network security thinking** — actively blocking three DNS bypass mechanisms (Firefox DoH, iCloud Private Relay, DDR) taught me that security isn't just about blocking threats, it's about controlling the paths traffic can take.
- **Reading and modifying config files** — working through `pihole.toml` to understand what each setting does and intentionally changing eight defaults to harden the setup.
- **Privacy by design** — setting privacy level 1 to hide domain names in logs, understanding the tradeoff between visibility and privacy, and making a deliberate choice rather than accepting defaults.
- **Troubleshooting real systems** — debugging port conflicts, permission errors reading config files, and SCP authentication issues reinforced that working with real infrastructure never goes exactly as documented.

---

## 📚 Resources

- [Pi-hole Documentation](https://docs.pi-hole.net/)
- [Pi-hole GitHub](https://github.com/pi-hole/pi-hole)
- [Firebog Blocklists](https://firebog.net/) — curated lists to expand coverage
- [Pi-hole Discourse Community](https://discourse.pi-hole.net/)

---

## 📄 License

This project documentation is open source. Pi-hole itself is licensed under the [EUPL](https://github.com/pi-hole/pi-hole/blob/master/LICENSE).
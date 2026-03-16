# 🛠️ Pi-hole Setup Guide

Complete installation and configuration guide for a network-wide Pi-hole ad blocker on Raspberry Pi.

---

## Prerequisites

Before starting, make sure you have:

- A Raspberry Pi (any model with network access — Pi 4 recommended)
- MicroSD card (16GB+ recommended, 32GB ideal)
- Raspberry Pi OS installed and booted
- A static IP address assigned to the Pi (via router DHCP reservation or manual config)
- SSH access or a connected keyboard/monitor
- Internet connection on the Pi

---

## Step 1 — Assign a Static IP

Pi-hole must have a **stable, unchanging IP address** so your router and devices can reliably send DNS traffic to it.

**Option A: DHCP Reservation (recommended)**

Log into your router admin panel and reserve a fixed IP for your Pi's MAC address. This is the easiest method and requires no changes on the Pi itself.

**Option B: Static IP on the Pi**

Edit `/etc/dhcpcd.conf`:

```bash
sudo nano /etc/dhcpcd.conf
```

Add at the bottom (adjust values for your network):

```
interface eth0
static ip_address=192.168.1.100/24
static routers=192.168.1.1
static domain_name_servers=127.0.0.1
```

Apply the changes:

```bash
sudo systemctl restart dhcpcd
```

---

## Step 2 — Install Pi-hole

Run the automated installer:

```bash
curl -sSL https://install.pi-hole.net | bash
```

The installer walks you through an interactive setup. Key choices during installation:

| Prompt | Recommended Setting |
|---|---|
| Upstream DNS provider | Google or Cloudflare (can change later) |
| Blocklists | Accept the default StevenBlack list |
| Admin web interface | Yes — install it |
| Query logging | Yes — enable it |
| Privacy mode | Show everything (for home use) |

When complete, the installer will display your admin panel URL and a temporary password. **Save the password.**

---

## Step 3 — Configure Upstream DNS Servers

After installation, open the web interface and go to **Settings → DNS**.

This setup uses:

- ✅ **Google (ECS, DNSSEC)** — `8.8.8.8` / `8.8.4.4`
- ✅ **Cloudflare (DNSSEC)** — `1.1.1.1` / `1.0.0.1`

![DNS Settings](screenshots/pihole-dns-upstream-servers.png)

ECS (Extended Client Subnet) allows geo-optimized responses from CDNs. DNSSEC validates DNS responses cryptographically to prevent spoofing.

> ⚠️ **Privacy note:** ECS sends partial client IP information to upstream resolvers. If privacy is a priority, consider using Cloudflare without ECS, or a local resolver like Unbound.

---

## Step 4 — Point Your Router's DNS to Pi-hole

Log into your router admin panel and set the **Primary DNS** to your Pi-hole's static IP address.

```
Primary DNS:   192.168.1.100   ← your Pi's IP
Secondary DNS: 8.8.8.8         ← fallback (optional)
```

Once saved, all devices on your network will automatically use Pi-hole for DNS without any per-device configuration.

> 💡 If you cannot change router DNS settings (e.g., ISP-locked routers), you can manually set DNS on individual devices instead.

---

## Step 5 — Access the Web Interface

Navigate to either:

- `http://pi.hole/admin`
- `http://YOUR_PI_IP/admin`

Log in with the password set during installation.

![Login Page](screenshots/pihole-login-page.png)

To change the password at any time:

```bash
pihole setpassword
```

---

## Step 6 — Update Gravity (Blocklists)

After installation, update the gravity database to pull in the latest blocklists:

```bash
sudo pihole -g
```

This setup currently blocks **81,131 domains** from the default blocklist.

To add more blocklists, go to **Lists** in the web interface and add URLs from sources like [Firebog.net](https://firebog.net/).

---

## Step 7 — Verify Everything Is Working

**Check Pi-hole status from the command line:**

```bash
pihole status
```

Expected output:
```
[✓] FTL is listening on port 53
    [✓] UDP (IPv4)
    [✓] TCP (IPv4)
    [✓] UDP (IPv6)
    [✓] TCP (IPv6)

[✓] Pi-hole blocking is enabled
```

**Check which ports are in use:**

```bash
sudo netstat -tulpn | grep pihole
```

Pi-hole FTL binds to:

| Port | Protocol | Purpose |
|---|---|---|
| 53 | UDP + TCP (IPv4 + IPv6) | DNS |
| 80 | TCP (IPv4 + IPv6) | HTTP web interface |
| 443 | TCP (IPv4 + IPv6) | HTTPS web interface |
| 67 | UDP | DHCP (if enabled) |
| 123 | UDP | NTP |

![Listening Ports](screenshots/pihole-listening-ports.png)

**Check version:**

```bash
pihole -v
```

```
Core version is v6.3
Web version is v6.4
FTL version is v6.4.1
```

---

## Step 8 — Enable as a Systemd Service (Auto-start)

Pi-hole FTL is registered as a systemd service automatically. Verify it's enabled to start on boot:

```bash
systemctl status pihole-FTL
```

Expected output shows `active (running)` and `enabled; preset: enabled`.

---

## Monitoring & Usage

### Dashboard

The main dashboard shows total queries, queries blocked, block percentage, and client activity over time.

![Dashboard](screenshots/pihole-dashboard-main.png)

### Query Log

The query log shows real-time DNS lookups from all clients. Blocked entries appear with a red stop icon; allowed entries pass through to upstream DNS.

![Query Log - Blocked](screenshots/pihole-query-log-blocked.png)
![Query Log - Internal](screenshots/pihole-query-log-internal.png)

### Useful CLI Commands

```bash
# Check status
pihole status

# Check version
pihole -v

# Update blocklists
sudo pihole -g

# Temporarily disable blocking (5 minutes)
pihole disable 5m

# Re-enable blocking
pihole enable

# Tail live query log
pihole tail

# Update Pi-hole
pihole update
```

---

## Keeping Pi-hole Updated

```bash
# Update Pi-hole core, web, and FTL
pihole update

# Update blocklists (run periodically or set a cron job)
sudo pihole -g
```

**Automated gravity updates via cron** — add to `/etc/cron.d/pihole` or use `crontab -e`:

```bash
# Update blocklists every Sunday at 2 AM
0 2 * * 0 root /usr/local/bin/pihole -g
```

---

## Troubleshooting

**Ads still showing after setup:**
- Confirm your device is using the Pi-hole IP for DNS: `nslookup google.com` should show the Pi's IP as the server
- Flush your device's DNS cache
- Check the query log to confirm queries are reaching Pi-hole

**Pi-hole not resolving DNS:**
- Run `pihole status` — confirm FTL is listening on port 53
- Check that no other service (like `systemd-resolved`) is occupying port 53: `sudo netstat -tulpn | grep :53`

**Web interface not loading:**
- Check `systemctl status pihole-FTL` for errors
- Confirm you're using the correct IP address

**A legitimate site is being blocked:**
- Search for the domain in the query log
- Click the entry and select **Whitelist** to allow it

---

## Resources

- [Pi-hole Documentation](https://docs.pi-hole.net/)
- [Pi-hole GitHub](https://github.com/pi-hole/pi-hole)
- [Firebog Blocklists](https://firebog.net/)
- [Pi-hole Discourse Community](https://discourse.pi-hole.net/)
- [Pi-hole Reddit](https://www.reddit.com/r/pihole/)
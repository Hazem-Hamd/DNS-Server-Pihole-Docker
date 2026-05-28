# 🍓 Pi-hole on Docker — Ubuntu Setup Guide

A self-hosted DNS sinkhole that blocks ads and trackers network-wide, running inside Docker on Ubuntu with a persistent configuration.

---

## 📸 Preview

> Pi-hole admin dashboard accessible at `http://<your-server-ip>:8081/admin`

---

## 📋 Prerequisites

- Ubuntu (tested on Ubuntu 22.04 / 24.04)
- Docker & Docker Compose installed
- A static local IP for your server (recommended)

---

## 🗂️ Project Structure

```
pihole/
├── docker-compose.yml
├── etc-pihole/        # Pi-hole config (auto-generated on first run)
└── etc-dnsmasq.d/     # dnsmasq config (auto-generated on first run)
```

---

## ⚙️ Step 1 — Fix Port 53 Conflict on Ubuntu

Ubuntu runs a built-in service called `systemd-resolved` that occupies port `53` by default.  
This will cause Docker to fail with: `Bind for 0.0.0.0:53 failed`.

Run these two commands to disable the stub listener:

```bash
sudo sed -r -i.orig 's/#?DNSStubListener=yes/DNSStubListener=no/g' /etc/systemd/resolved.conf
sudo systemctl restart systemd-resolved
```

---

## 📁 Step 2 — Create the Project Folder

```bash
mkdir pihole && cd pihole
```
<img width="580" height="175" alt="Screenshot 2026-05-28 233758" src="https://github.com/user-attachments/assets/3cffeb19-486a-400c-b879-922f0f3ff83b" />

---

## 🐳 Step 3 — docker-compose.yml

Create the compose file:

```bash
nano docker-compose.yml
```

Paste the following configuration:

```yaml
version: "3"

services:
  pihole:
    container_name: pihole
    image: pihole/pihole:latest
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "8081:80/tcp"
    environment:
      TZ: 'Africa/Cairo'
      WEBPASSWORD: 'admin'
    volumes:
      - './etc-pihole:/etc/pihole'
      - './etc-dnsmasq.d:/etc/dnsmasq.d'
    restart: unless-stopped
```
<img width="1109" height="622" alt="Screenshot 2026-05-28 233819" src="https://github.com/user-attachments/assets/99fc4c1e-6b22-47e5-b700-1c94443f4544" />

> **Notes:**
> - Port `8081` is used instead of `80` to avoid conflicts with other services (e.g., Coolify).
> - `TZ` is set to `Africa/Cairo` — change to your timezone if needed.
> - Change `WEBPASSWORD` to something secure before deploying!

Save and exit: `Ctrl+O` → `Enter` → `Ctrl+X`

---

## 🚀 Step 4 — Start Pi-hole

```bash
sudo docker compose up -d
```

---

## 🔐 Step 5 — Set / Reset Admin Password

```bash
sudo docker exec -it pihole pihole setpassword admin
```

Replace `admin` with your desired password.

---

## 🌐 Step 6 — Access the Dashboard

Open your browser and navigate to:

```
http://<your-server-ip>:8081/admin
```
<img width="1328" height="992" alt="Screenshot 2026-05-28 234003" src="https://github.com/user-attachments/assets/21e0f6a7-24cd-481a-8ac8-645ddf295b96" />

Default credentials: `admin` (or whatever password you set above).

---

## 🛑 Useful Docker Commands

| Action | Command |
|---|---|
| Start Pi-hole | `sudo docker compose up -d` |
| Stop Pi-hole | `sudo docker stop pihole` |
| Restart Pi-hole | `sudo docker restart pihole` |
| View logs | `sudo docker logs pihole` |
| Enter container shell | `sudo docker exec -it pihole bash` |

---

## 🔁 Pointing Devices to Pi-hole DNS

Once running, set your router's primary DNS server to your server's local IP (e.g., `192.168.1.100`).  
This routes all DNS queries through Pi-hole for network-wide ad blocking.

---

## 📦 Volumes (Persistent Data)

| Host Path | Container Path | Purpose |
|---|---|---|
| `./etc-pihole` | `/etc/pihole` | Pi-hole settings & block lists |
| `./etc-dnsmasq.d` | `/etc/dnsmasq.d` | DNS/DHCP config |

Data persists across container restarts and updates.

---

## 🧩 Compatibility Notes

- Tested with **Coolify** on the same host — port `8081` avoids the default port `80` conflict.
- If you use another reverse proxy (Nginx, Caddy, Traefik), you can map Pi-hole behind a subdomain and remove the `8081` port mapping.

---

## 📄 License

MIT — feel free to fork and adapt.

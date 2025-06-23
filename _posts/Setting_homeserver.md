---
layout: post
title: "Do You Really Need a Home Server? Exploring VPS as a Home Lab Alternative"
date: 2025-06-23
categories: [homelab, vps, cloud, self-hosting]
tags: [server, cloud, vps, docker, jellyfin, traefik, authelia, hetzner, duckdns]
author: "Your Name"
excerpt: "Is a physical home server necessary? Discover how a VPS can replace your home server for media streaming, automation, and more—complete with Docker, Traefik, and secure authentication."
toc: true
math: false
---

> **Let me ask you something:**  
> Do you actually need a home server?  
> And I don’t mean it in a “you don’t need it, you just want it” kind of way.  
> I mean, does it have to physically be in your house?

---

## The Case for a "Home" Server That Isn't at Home

The idea of a home server not actually being at home is nothing new. [Techno Tim](https://www.youtube.com/c/TechnoTim) has already moved servers into a data center rack. But for most of us, colocation is expensive and inconvenient. For example, renting two rack units in a German data center can cost **42€ per month** (plus electricity and a 3-year contract), not to mention hardware and travel costs.

But don’t lose hope! You can set up a home server in the cloud without breaking the bank. Enter: the humble **Virtual Private Server (VPS)**.

---

## Why Use a VPS as a Home Server?

Many of us already use VPSes for things like VPNs, websites, or chat servers. But can you use a VPS as a full-featured home server—running Jellyfin, Plex, Navidrome, the Arr Suite, Paperless, or Immich?

Let's find out!

---

## 💡 Sponsored by Brilliant.org

> **Brilliant** is where you learn by doing, with thousands of interactive lessons in math, data analysis, programming, and AI.  
> Their course ["How LLMs Work"](https://brilliant.org/) is a great entry point if you want to learn about the tech powering modern AI.  
> Try Brilliant free for 30 days, and get 20% off an annual premium subscription as my reader!

---

## VPS vs. Physical Hardware

### Pros:
- **No upfront hardware cost** (e.g., Raspberry Pi 5 kit ≈ 120€)
- **No commitment**—try self-hosting for a few euros/month
- **Accessible from anywhere** (great for digital nomads or students)

### Cons:
- **Limited storage** (most cheap VPSes: 15–40GB)
- **No GPU/transcoding** (not ideal for 4K streaming)
- **Bandwidth limits** (depends on your home and VPS connection)

---

## Affordable VPS + Storage: The Hetzner Example

- **Hetzner CX22 VPS:** 2 vCPUs, 4GB RAM, 40GB SSD, IPv4 — **4.51€/mo**
- **Hetzner Storage Box BX11:** 1TB for **3.81€/mo**
- **Total:** **8.32€/mo** (cheaper than Netflix!)

Or, try [liteserver.nl](https://liteserver.nl/) for 1 CPU, 1GB RAM, 512GB storage at 6€/mo.

---

## Limitations

- **Storage:** 1TB is enough for a modest collection, but not for hoarders.
- **Transcoding:** No GPU, so stick to direct play and compressed FullHD/HD content.
- **Bandwidth:** VPSes often have gigabit connections, but your home internet may be the bottleneck.

---

## Legal Note

> **No piracy here!**  
> Torrenting is not illegal, but downloading copyrighted content is.  
> We'll use public domain or copyleft media for this guide.

---

## Step-by-Step: Setting Up Your VPS Home Server

### 1. **Buy Your VPS and Storage Box**

- Register at [console.hetzner.cloud](https://console.hetzner.cloud/)
- Buy a Storage Box at [robot.hetzner.com](https://robot.hetzner.com/)
- Choose the cheapest options (BX11 for storage, CX22 for VPS)

### 2. **Create and Secure Your Server**

- Generate and add your SSH key
- Set up a temporary firewall to allow only your IP
- SSH into your server and update it:
  ```sh
  apt update && apt upgrade
  apt install sudo cifs-utils curl
  ```

- Create a non-root user and set up SSH access:
  ```sh
  useradd -m -s /bin/bash -G sudo youruser
  passwd youruser
  ssh-copy-id -i ~/.ssh/yourkey.pub youruser@your-vps-ip
  ```

- Harden SSH:
  - Edit `/etc/ssh/sshd_config`:
    ```
    PasswordAuthentication no
    PermitRootLogin no
    ```
  - Restart SSH: `sudo systemctl restart sshd`

### 3. **Mount Your Storage Box**

- Enable Samba on your Storage Box via Hetzner's panel
- Create `.smb` credentials file in your home directory:
  ```
  user=your-storagebox-user
  password=your-storagebox-password
  domain=server
  ```
- Restrict permissions: `chmod 600 ~/.smb`
- Create mount point: `sudo mkdir -p /mnt/media`
- Edit `/etc/fstab`:
  ```
  //your-storagebox-url /mnt/media cifs uid=0,credentials=/home/youruser/.smb,iocharset=utf8,noperm 0 0
  ```
- Mount:  
  ```sh
  sudo systemctl daemon-reload
  sudo mount -a
  df -h
  ```

### 4. **Install Docker**

- Follow [Docker's official instructions](https://docs.docker.com/engine/install/debian/)
- Add your user to the docker group:
  ```sh
  sudo usermod -aG docker youruser
  newgrp docker
  docker run hello-world
  ```

### 5. **Get a Domain Name**

- Use [DuckDNS](https://duckdns.org/) for a free domain
- Update your IP in DuckDNS

### 6. **Clone and Configure Your Compose Stack**

- Clone your prepared [GitHub repo](https://github.com/yourusername/yourrepo)
- Copy `.env.template` to `.env` and fill in your DuckDNS domain/token
- Review and edit `compose.yaml` for your services (Jellyfin, Sonarr, Radarr, Deluge, Traefik, Authelia, etc.)

### 7. **Start Your Services**

```sh
docker compose up -d --build
```

- Check Traefik logs: `docker logs -f traefik`
- Wait for Let's Encrypt certificates to be issued

### 8. **Secure with Authelia**

- Hash your password and edit `/var/opt/data/authelia/users_database.yml`
- Restart Authelia: `docker restart authelia`
- Test authentication in incognito/private browser windows

---

## Service Configuration Tips

- **Traefik**: Use Docker labels for automatic reverse proxying and SSL
- **Authelia**: Protect all services except those with their own auth (e.g., Jellyfin)
- **Jellyfin**: Set up libraries pointing to `/mnt/media/tv` and `/mnt/media/movies`
- **Direct Play**: Use clients that support direct playback for best performance

---

## Testing and Final Steps

- Remove your temporary firewall after confirming all services are protected
- Test each service with `curl` to ensure authentication is enforced
- Monitor resource usage (`htop`, `docker stats`)—most self-hosted apps are lightweight!

---

## Conclusion

A VPS can be a fantastic, affordable alternative to a physical home server for most self-hosted services—especially if you value flexibility and low upfront cost.  
If you need more power, Hetzner and others let you scale up easily.

---

## Further Reading & Resources

- [Hetzner Cloud Docs](https://docs.hetzner.com/)
- [Docker Compose](https://docs.docker.com/compose/)
- [Traefik](https://doc.traefik.io/traefik/)
- [Authelia](https://www.authelia.com/)
- [DuckDNS](https://www.duckdns.org/)
- [Brilliant.org](https://brilliant.org/)

---

*Thanks for reading! If you found this helpful, share your thoughts in the comments and consider subscribing for
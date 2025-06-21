---
title: "How to Set Up Dynamic DNS (DDNS) with Cloudflare and a Raspberry Pi"
date: 2025-06-21
categories: [networking, cloudflare, ddns, homelab]
tags: [raspberry-pi, linux, automation, bash, cloudflare, dns]
author: "Ouassim Kiassa"
excerpt: "Learn how to automate your home lab's DNS updates using Cloudflare's API and a Raspberry Pi or any Linux machine. Never worry about your changing IP address again!"
toc: true
---

> ☕️ **Get your coffee ready!** This guide will show you how to set up Dynamic DNS (DDNS) for your home lab using a Raspberry Pi (or any Linux machine) and Cloudflare's API. It's quick, essential, and will save you tons of time!

---

## What is Dynamic DNS (DDNS) and Why Do You Need It?

If you run servers or services at home, you probably want to access them remotely using a domain name. The problem? Most home internet connections have a **dynamic IP address**—it changes periodically. Every time it changes, your domain points to the wrong place!

> **Static IP?** If you can pay for one, that's the best solution. But if not, DDNS is your friend.

With DDNS, a script running inside your home network automatically updates your DNS record whenever your public IP changes. No more manual updates!

---

## Prerequisites

1. **A domain name**  
   *Cloudflare does not support some free domains (like `.tk`, `.ga`). Use a standard TLD like `.com`, `.io`, etc.*

2. **A free Cloudflare account**  
   [Sign up here](https://dash.cloudflare.com/sign-up)

3. **A Linux machine**  
   A Raspberry Pi is perfect, but any always-on Linux box will do.

---

## Step 1: Prepare Your Raspberry Pi (or Linux Machine)

Open your terminal and SSH into your device.

```sh
ssh pi@your-pi-ip
```

Update your system and install `git`:

```sh
sudo apt update && sudo apt install git -y
```

---

## Step 2: Clone the Cloudflare DDNS Script

Find a good [Cloudflare DDNS updater script](https://github.com/K0p1-Git/cloudflare-ddns-updater) or use the template below.

```sh
git clone https://github.com/K0p1-Git/cloudflare-ddns-updater.git
cd cloudflare-ddns-updater
cp cloudflare-template.sh cloudflare.sh
```

---

## Step 3: Configure the Script

Open the script in your favorite editor:

```sh
nano cloudflare.sh
```

Update the following fields with your Cloudflare account info:

```bash
# ...existing code...
auth_email="your-cloudflare-email@example.com"
auth_key="your-global-api-key"
zone_identifier="your-zone-id"
record_name="your.domain.com"
# ...existing code...
```

### How to Find Your Cloudflare API Key and Zone ID

1. **API Key:**  
   - Go to [Cloudflare Dashboard](https://dash.cloudflare.com/)
   - Click your profile (top right) → **API Tokens**
   - Under "API Keys", click **View** next to "Global API Key"
   - Complete the CAPTCHA and copy your key

2. **Zone ID:**  
   - In your Cloudflare dashboard, select your domain
   - Scroll down on the Overview page to find your **Zone ID**

3. **Record Name:**  
   - This is the DNS record you want to update (e.g., `home.yourdomain.com`)

---

## Step 4: Test the Script

Make the script executable:

```sh
chmod +x cloudflare.sh
```

Run it:

```sh
./cloudflare.sh
```

Check your DNS record in Cloudflare. If it changed to your current public IP, **success!**

---

## Step 5: Automate with Cron

Edit your crontab to run the script every minute:

```sh
crontab -e
```

Add this line (replace the path as needed):

```cron
*/1 * * * * /bin/bash /path to your script
```

> This will check and update your DNS every minute.

---

## Step 6: (Optional) Handle Multiple Subdomains

You can use CNAME records in Cloudflare to point multiple subdomains to your main DDNS record.  
For example, set `test.yourdomain.com` as a CNAME to `home.yourdomain.com`.


---

## Final Thoughts

This setup is **essential** for any home lab. With a Raspberry Pi and Cloudflare, your domain will always point to your home—even if your IP changes.  
If you found this helpful, please share, comment, and subscribe!

---

## Resources

- [Cloudflare DDNS Updater Script (GitHub)](https://github.com/K0p1-Git/cloudflare-ddns-updater)
- [Cloudflare API Documentation](https://api.cloudflare.com/)

---

*Happy hacking! 🚀*
---
title: "Vaultwarden On Raspberry Pi"
seoTitle: "How to Self‑Host Vaultwarden on a Raspberry Pi Using Cloudflare Tunnel"
seoDescription: "A clear, beginner‑friendly guide to installing Vaultwarden on a Raspberry Pi with Docker Compose and securing it through Cloudflare Tunnels. Includes setup."
datePublished: Sun Jan 04 2026 22:50:52 GMT+0000 (Coordinated Universal Time)
cuid: cmk0bsaa2000502l1348ehjfv
slug: vaultwarden-on-raspberry-pi
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/Jj3-Wh4MHs4/upload/d25c4c53dec7356705295244c42e6dab.jpeg
tags: self-hosted, password-manager, vaultwarden

---

Self‑hosting your own password manager is one of the most empowering projects you can take on, especially if you are already running services on a Raspberry Pi. For this setup, I decided to host an open-source version of Bitwarden using Vaultwarden. My goal is to give you a clean, straightforward guide that reflects what actually worked for me, including the mistakes I made and the lessons I learned.

Before we get into the steps, here is a quick look at how I got started.

## How I Found Vaultwarden

I first came across the Vaultwarden repo sometime in the summer of 2025. After watching a YouTube tutorial, I figured it was something I could handle, even though I was still pretty new to Docker. I tried running the script directly from the README:

```yaml
docker pull vaultwarden/server:latest
docker run --detach --name vaultwarden \
  --env DOMAIN="https://vw.domain.tld" \
  --volume /vw-data/:/data/ \
  --restart unless-stopped \
  --publish 127.0.0.1:8000:80 \
  vaultwarden/server:latest
```

It worked in the sense that I could see the login screen when I navigated to the resource and port. But I could not actually log in or access anything, as you can see here:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1767486027438/24517ed4-11a1-4fd8-aeb2-73407dc96b0d.png align="center")

I searched around for a while but did not have the time to dig deeper. I eventually moved on to other projects like Portainer and SterlingPDF.

## Prerequisites

Before you start, here is what you need:

* Raspberry Pi running Raspberry Pi OS
    
* Docker and Docker Compose installed
    
* A Cloudflare account with a registered domain
    
* Basic familiarity with SSH
    
* A willingness to troubleshoot a bit
    

This setup is beginner friendly, but having these pieces in place will save you time.

## Returning to the Project

I came back to this project at the end of 2025 around New Year's weekend. This time, I switched to Docker Compose and cleaned up the configuration. Here is the actual file I used:

```yaml
services:
  vaultwarden:
    image: vaultwarden/server:alpine
    container_name: vaultwarden
    restart: always
    environment:
      DOMAIN: "https://pass.example.com"
      SIGNUPS_ALLOWED: "true" # Set to false after creating your account
      ADMIN_TOKEN: ${ADMIN_TOKEN}
    volumes:
      - ./vw-data:/data
    ports:
      - 11001:80
```

### Why Alpine Matters on Raspberry Pi

This is one of the biggest gotchas. Raspberry Pi OS uses ARM architecture. The `latest` Vaultwarden image is built for x86 systems, so it will not run correctly on a Pi. The Alpine variant supports ARM out of the box, which is why `vaultwarden/server:alpine` is required.

### Why HTTPS Is Required

Vaultwarden does not support HTTP for normal operation. It needs to know it is behind HTTPS so that attachments, WebSockets, and the web vault function correctly. If you try to access it over plain HTTP, you will run into the same issues I did during my first attempt.

### Running the Container

I started the container with:

```yaml
docker compose up -d
```

And checked logs with:

```yaml
docker logs -f vaultwarden
```

This step is optional but helpful for catching errors early.

## Why I Chose Cloudflare Tunnels

Most tutorials use Caddy or Nginx Proxy Manager. I already had a Cloudflare Tunnel running for my NocoDB project and liked the simplicity of outbound-only connections. No port forwarding, no exposing your home network, and Cloudflare handles the heavy lifting.

In my first project, I installed the Cloudflare Tunnel via the CLI. This time, I wanted to use the Cloudflare One Dashboard. My domain was already registered with Cloudflare, so I logged into Cloudflare One.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1767562423389/fa88030c-5ed6-4bff-a772-72aa8e5d9f96.png align="center")

In the Zero Trust Platform, click on "Networks".

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1767578358422/f5dffaff-2094-4e65-b2cd-fceaff8d0070.jpeg align="center")

Under Networks, go to “Manage Tunnels” and set up a tunnel if you do not already have one. The process is straightforward.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1767564757437/2669c9d4-f8c3-46bf-af24-839abde83451.png align="center")

From here, manage the routed hostnames as shown below:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1767564999285/1369c1ab-9ffe-4409-a79e-f0958edae4f5.png align="center")

Cloudflare will automatically create a CNAME record in your DNS settings. Within a few seconds, you will be able to navigate to your Vaultwarden instance securely.

This method works for almost any service running on a port. I prefer it over opening ports on my router, which can introduce security risks. Cloudflare Tunnels create an outbound-only connection from your device to Cloudflare and rely on their global network to handle the rest.

## Backup Reminder

Your entire Vaultwarden instance lives in the `/data` directory. Back it up regularly. A simple file-level backup is enough to restore your vault if anything goes wrong.

## Security Tips

A few quick reminders:

* Disable signups after creating your account.
    
* Store your `ADMIN_TOKEN` in an `.env` file instead of hardcoding it.
    
* Keep your Pi updated.
    
* Use strong passwords for your Cloudflare account and enable 2FA.
    

Small steps like these go a long way.

## Troubleshooting Tips

Here are a few common issues you might run into:

* Using the wrong image on Raspberry Pi
    
* DOMAIN not set or set incorrectly
    
* Tunnel routing to the wrong port
    
* Trying to access Vaultwarden over HTTP
    
* Missing or incorrect volume paths
    

Checking logs usually points you in the right direction.

## What’s Next

I will create another post soon about generating the keys needed to securely enable the Admin page. I also plan to cover backups, token rotation, and a few other Vaultwarden best practices.

If you are self-hosting other tools on your Pi, Cloudflare Tunnels can simplify your entire setup. This approach has been reliable for me, and I hope this guide helps you get your own instance running smoothly.
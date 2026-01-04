---
title: "Nocodb on Raspberry Pi"
seoTitle: "Nocodb Setup on Raspberry Pi Guide"
seoDescription: "Learn how to set up NocoDB on a Raspberry Pi using Docker, Cloudflare Tunnel, and R2 for remote access and file storage"
datePublished: Fri Jul 04 2025 04:01:31 GMT+0000 (Coordinated Universal Time)
cuid: cmcoaf6y3000f02jo17p8a37d
slug: nocodb
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/ZfVyuV8l7WU/upload/ab10fe3e4d1771682dd9debc1327a282.jpeg
tags: opensource, databases, beginners, s3-bucket

---

### Intro

Recently, I discovered Nocodb while searching for alternatives to AirTable. I needed a solution for a work-related problem and initially created a database on AirTable. However, I was hesitant to pay for it when a great open-source alternative might be available. Nocodb emerged as a top choice, leading me to transition away from AirTable. I soon realized that Nocodb could also be useful for a personal project, which I will guide you through here.

### Setup

I used [Docker](https://www.docker.com/) on my Raspberry Pi 5 to install it with SQLite.

```bash
docker run -d --name nocodb \
-v "$(pwd)"/nocodb:/usr/app/data/ \
-p 8080:8080 \
nocodb/nocodb:latest
```

After successful installation, I ran the command `docker start nocodb` to get access to the database. I used the `<pi-ip-address:8080>`, created a master username and password, and then created the initial tables and schema. The Initial setup was straightforward.

![Local Access](https://cdn.hashnode.com/res/hashnode/image/upload/v1751677709262/8f23ef0f-d744-4204-b306-58761c1dc7ad.png align="center")

After about a day of playing with the database on my local network, I quickly realized that I might need to access this outside my network. What if I’m on the road and want to add something to my database? Or, what if I wanted to invite someone else to add stuff to my database?

### Remote access

I had a few options: port forwarding, VPN, hosting on the cloud, or Cloudflare Tunnel. I didn’t want to do port forwarding for a few reasons. For one, I’m not keen on exposing ports on my home network; it seems like a bad idea (though I may be wrong here). VPN wasn’t an option either, as it was cost-prohibitive. My goal for this setup was to keep things free if possible, and I wanted to host everything on my Raspberry Pi myself. That left [Cloudflare Tunnel](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/).

Configuring Cloudflare Tunnel was surprisingly straightforward with the help of ChatGPT and online guides. I’m already a big fan of [Cloudflare](https://www.cloudflare.com/), and since my dev domain is hosted there, integrating the tunnel was even easier.

## Setting Up Cloudflare Tunnel

I installed the [`cloudflared`](https://github.com/cloudflare/cloudflared) daemon on my Raspberry Pi, then authenticated it with my Cloudflare account using:

```bash
cloudflared tunnel login
```

After creating the tunnel and configuring it to forward traffic to NocoDB’s local port (8080), I set up a DNS CNAME record on Cloudflare to point my subdomain to the tunnel endpoint.

This allowed me to securely access NocoDB remotely via HTTPS without exposing ports on my home network. On top of that, I configured the Raspberry Pi firewall [`ufw`](https://help.ubuntu.com/community/UFW) to only allow SSH traffic, blocking everything else. Inside NocoDB, I created user accounts with appropriate roles to restrict access as needed.

![Accessing via domain](https://cdn.hashnode.com/res/hashnode/image/upload/v1751677858605/c1557224-38d0-4b70-8ff4-487d2834c395.png align="center")

## Integrating Cloudflare R2 for File Storage

I wanted to offload file uploads (images, documents) from my Pi to scalable cloud storage. Cloudflare R2’s S3-compatible API made it a natural fit.

I configured NocoDB’s Docker container with environment variables pointing to my R2 bucket:

```dockerfile
-e S3_ACCESS_KEY_ID=<your-access-key-id> \
-e S3_SECRET_ACCESS_KEY=<your-secret-access-key> \
-e S3_ENDPOINT=https://<account-id>.r2.cloudflarestorage.com \
-e S3_BUCKET=<your-bucket-name> \
-e S3_REGION=auto \
-e S3_FORCE_PATH_STYLE=true
```

Uploads now go directly to R2, keeping my Pi’s storage free and scalable.

## Backups and Automation

Before moving file storage to R2, I backed up my local SQLite database and uploaded folders. I used [`rclone`](https://rclone.org/) to sync my uploads folder to R2 manually, and then create a simple shell script to automate this process.

Scheduling that script with a cron job on the Pi ensured my uploads stay in sync without manual effort:

```bash
0 2 * * * /home/pi/sync-nocodb-uploads.sh >> /home/pi/cron-sync.log 2>&1
```

### Conclusion

If you’re looking for an affordable, flexible way to run your own database accessible anywhere, give NocoDB on a Raspberry Pi with Cloudflare Tunnel and R2 a shot!
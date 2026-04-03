---
title: "Post 8: Cloudflare Tunnel"
slug: post-8
date_published: 2025-05-26T21:20:00.000Z
date_updated: 2025-06-18T18:17:24.000Z
tags: Kubernetes
categories: [kubernetes]
---

---

I had my website internally hosted within a docker swarm. for me to be successful in this cabernets set up I needed to start clean. This means deleting any existing tunnels and dns entries for the site name you want to use.

You can go ahead and create your web site on a fixed ip within the metallb range.

After deploying our WordPress site on our k3s cluster, it's time to expose it securely to the internet using Cloudflare Tunnel. This approach avoids opening ports on your router and keeps your services protected behind Cloudflare's network.

---

## Goals

- Secure public access to WordPress
- Use Cloudflare Tunnel to avoid port forwarding
- Assign the domain `stevenfmeyer.me` to the WordPress service running at MetalLB IP `10.0.0.35`

---

## Step 1: Prep Your Cloudflare Account

1. Log in to Cloudflare and ensure `stevenfmeyer.me` is added to your dashboard.
2. Disable any existing A, AAAA, or CNAME records for the root domain to avoid conflicts.

---

## Step 2: Install `cloudflared` on a Jumpbox or LXC

We’ll run the tunnel from a centralized container or node.

    sudo apt update && sudo apt install cloudflared -y
    

---

## Step 3: Create the Tunnel

1. Authenticate with Cloudflare:

    cloudflared tunnel login
    

Follow the instructions to log into your cloudflare instance and click on the account to authenticate.

1. Create the tunnel:

    cloudflared tunnel create wordpress-tunnel
    

1. This will generate a credentials JSON file, usually in `/root/.cloudflared/`.

---

## Step 4: Create a Config File

Save this as `/root/.cloudflared/config.yml`:

    tunnel: wordpress-tunnel
    credentials-file: /root/.cloudflared/<tunnel-id>.json
    
    ingress:
      - hostname: stevenfmeyer.me
        service: http://10.0.0.35
      - service: http_status:404
    

Replace <tunnel-id> with the id of the json file in the cli output.

---

## Step 5: Route the Domain

Create a CNAME in Cloudflare DNS:

    cloudflared tunnel route dns wordpress-tunnel stevenfmeyer.me
    

---

## Step 6: Run the Tunnel

To run manually:

    cloudflared tunnel run wordpress-tunnel
    

Or persist with systemd:

    cloudflared service install
    

---

## Done!

Your WordPress site at `stevenfmeyer.me` is now securely accessible without opening a single port.

In the next post, we'll cover setting up HTTPS, redirects, and Cloudflare Zero Trust policies.

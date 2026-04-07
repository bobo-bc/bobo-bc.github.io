---
title: Configure Network for homelab
layout: homelab
series: infrastructure
order: 3
tags: [series-infrastructure, network]
categories: [homelab]
---
# Building a Home Network for Your Homelab

When you first start a homelab, everything usually lives on the same network — your laptop, your smart TV, your servers, all mixed together. It works… until it doesn’t.

At some point, things get messy:
	•	Devices change IP addresses
	•	Services become hard to reach
	•	Smart home gear starts talking to everything

A good homelab network fixes this by focusing on three things:
	•	Separation – keeping different types of devices apart
	•	Predictability – knowing where everything lives
	•	Usability – accessing services by name instead of IP

This article walks through a simple, real-world setup you can adapt. Don’t worry about copying it exactly — understand the structure, then make it your own.

⸻

## The Big Picture

Here’s the structure I use at home:
	•	A gateway router that supports VLANs
	•	Three separate networks (homelab, trusted devices, IoT)
	•	Each network has its own IP range
	•	A simple DNS setup so everything is reachable by name

At a high level, internet traffic flows like this:

```
ISP Modem → Gateway → VLANs → Your Devices
```
My ISP requires their modem to stay in place (it handles both internet and TV), so it lives at 192.168.0.1. My gateway sits behind it and handles everything inside the network.

⸻

## A Proper Gateway (and Why It Matters)

For a long time, I used the router provided by my ISP. That’s completely fine when you’re getting started — you can reserve IP addresses and run a few services without issue.

The real upgrade came when I added a dedicated gateway (in my case, a Unifi Gateway). The key benefit wasn’t speed — it was control.

Specifically, it allowed me to:
	•	Create multiple isolated networks (VLANs)
	•	Control how those networks talk to each other
	•	Properly organize devices by role

If you’re building a homelab, this is the point where things start to feel intentional instead of improvised.

⸻

### How My Network Is Structured
![home_network_diagram.svg](/assets/img/my_networkpng.drawio.png)


I run three separate networks. Each one has a clear purpose.

⸻

#### Homelab Network — 10.0.0.0/24

This is where the important stuff lives:
	•	Proxmox hosts
	•	Infrastructure services
	•	Networking gear (like access points)

Anything I actively manage goes here.

This network is protected from IoT devices, which are often noisy or poorly secured.

⸻

#### Trusted Device Network (T-net) — 10.0.1.0/24

This is for personal devices:
	•	Laptops
	•	Phones
	•	Tablets

These are devices I trust, so this network is allowed to:
	•	Access the homelab
	•	Access IoT devices
	•	Access the internet

This is what lets you control your smart home from your phone without jumping through hoops.

⸻

#### IoT Network (I-net) — 10.0.2.0/24

This is where everything else goes:
	•	Smart TVs
	•	Streaming devices
	•	Smart plugs and bulbs
	•	Robot vacuums

These devices get:
	•	Internet access
	•	No access to other networks

If one gets compromised, it can’t reach anything important.

⸻

#### Firewall Rules (Plain English)
	•	IoT → Internet only
	•	Trusted → Everything
	•	Homelab → No IoT access

That’s it. Simple and predictable.

⸻

## Managing IP Addresses (Keeping Things Predictable)

Each network (/24) gives you about 254 usable addresses. Instead of letting things get random, I split that space into two zones:
	•	.1 – .100 → reserved for manual assignment
	•	.101 – .254 → DHCP pool (automatic addresses)

This creates a really useful pattern:
	•	Important devices always live in the same range
	•	Random devices never collide with them

💡 Tip: Make IPs truly stick
Reserve the IP in your router and set the same static IP on the device itself.
That way, even if you reset your router, nothing breaks.

⸻

#### A Quick Note on What /24 Means

You’ll see network ranges written like 10.0.0.0/24.

All that really means is:
	•	It’s a block of addresses
	•	It contains ~254 usable IPs
	•	It acts like its own small “neighbourhood”

You can make smaller networks, but for a homelab, sticking with /24 keeps things simple and easy to reason about.

⸻

#### Placing Services Where They Belong

Not everything belongs on the “homelab” network.

Two of my services intentionally live on the IoT network:
	•	Home Assistant (10.0.2.4)
	•	Plex Media Server (10.0.2.10)

Why?

Because they primarily talk to IoT devices:
	•	Home Assistant manages them
	•	Plex serves media to TVs and streaming devices

This is an important mindset shift:

Place services based on what they need to talk to, not just where they “feel like they belong.”

⸻

## Making Your Services Easy to Reach (DNS)

Typing IP addresses gets old fast:

http://10.0.0.67

Instead, you want:

http://sonarr.home

That’s where DNS comes in.

⸻

#### The Basics

There are two record types you’ll use most:

A Record → maps a name to an IP

traefik.home → 10.0.0.67

CNAME → maps a name to another name

sonarr.home → traefik.home
radarr.home → traefik.home


⸻

#### Reverse Proxy (Traefik)

A reverse proxy like Traefik sits at a single IP and routes traffic based on hostname.

That means:
	•	One A record (Traefik)
	•	Many CNAMEs (your services)

Traefik figures out where each request should go.

This keeps your DNS clean and scalable.

⸻

#### DNS Options

There are a couple of easy ways to run this:
	•	Pi-hole — simple, popular, and doubles as an ad blocker
	•	Your gateway (like Unifi) — often has built-in DNS features

Both work well. Use whatever fits your setup.

⸻

## Putting It All Together

At this point, everything works as a system:
	•	The gateway separates your network into VLANs
	•	Firewall rules control how they interact
	•	IP ranges keep devices predictable
	•	DNS makes everything easy to access

Adding something new becomes straightforward:
	•	New server → assign IP in 10.0.0.x
	•	New IoT device → connect to IoT network
	•	New service → add a DNS entry

The structure is already there — you’re just filling it in.

⸻

## Final Thoughts

You don’t need a perfect network to get started.

Start simple:
	•	One network
	•	A few reserved IPs

Then grow into:
	•	VLANs
	•	Better IP organization
	•	DNS and reverse proxies

The goal isn’t complexity — it’s clarity.

If you can answer:
	•	“Where does this device belong?”
	•	“What should it be allowed to talk to?”

…then your network is doing exactly what it should.

⸻

This article is part of a series. See also: Proxmox Build and PBS Backups for the rest of the homelab stack.

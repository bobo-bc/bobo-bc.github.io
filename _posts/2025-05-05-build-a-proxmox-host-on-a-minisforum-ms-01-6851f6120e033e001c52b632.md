---
title: Build a Proxmox Host on a Minisforum MS-01
slug: build-a-proxmox-host-on-a-minisforum-ms-01-6851f6120e033e001c52b632
date_published: 2025-05-05T23:43:18.000Z
date_updated: 2025-05-05T23:43:18.000Z
tags: Homelab, Kubernetes, #Migrated-1750201873783, #wp, #wp-post, #Import 2025-06-17 16:11
---

---

*Published by bobo on May 5, 2025*

If you’re looking to create a compact yet beastly Proxmox server, the **Minisforum MS-01** is a perfect candidate. In this post, I’ll show you how to turn one into a full-featured **Proxmox VE host** with:

✅ **96 GB of RAM**

✅ **20-core CPU**

✅ **1TB NVMe SSD dedicated to Proxmox**

✅ **Bonded 2.5GbE ports with a static IP**

✅ **LACP teaming for performance + redundancy**

---

## **? What You’ll Need**

- **Minisforum MS-01** (or similar with dual NICs)
- Proxmox VE ISO → [Download it here](https://www.proxmox.com/en/downloads)
- USB flash drive (2GB+)
- Monitor + keyboard for first boot
- Static IP address: 10.0.0.5
- Your router/switch must support **802.3ad (LACP)** for bonding

---

## **? Step 1: Flash the Proxmox ISO**

1. Download the latest **Proxmox VE ISO**.
2. Use [balenaEtcher](https://etcher.io/) or [Rufus](https://rufus.ie/) to flash the ISO to your USB.
3. Safely eject the drive when done.

---

## **? Step 2: Install Proxmox on the MS-01**

1. Plug the USB drive into your MS-01 and boot it.
2. Enter BIOS (DEL or F11) and boot from USB.
3. Choose **Install Proxmox VE**.
4. Select the **1TB NVMe** as the install target.
- Use **ext4** or **ZFS RAID0** (if you’ll add more drives later).

5. Set your root password and email.
6. Skip static IP for now — we’ll handle that post-install.
7. Finish the install, then reboot and remove the USB.

---

## **? Step 3: Bond the Dual 2.5GbE NICs + Set Static IP**

We’ll configure **LACP (802.3ad) bonding** to combine your two NICs into a single, fault-tolerant link with increased throughput.

### **1. Log into the Proxmox Web UI**

Visit: https://your-proxmox-ip:8006

(Log in with root and the password you set.)

If you don’t know the IP, check it via console:

    ip a

---

### **2. Create a Bond Interface**

1. Go to **Datacenter > your-node > System > Network**.
2. Click **Create > Linux Bond**.
- Name: bond0
- Slaves: enp1s0, enp2s0 (check ip a for your NIC names)
- Mode: 802.3ad (LACP)
- Hash Policy: layer2+3
- Autostart: ✅

Click **Create**.

---

### **3. Create a Linux Bridge for VMs/Containers**

1. Click **Create > Linux Bridge**.
- Name: vmbr0
- Bridge Ports: bond0
- IPv4: Static
- IP: 10.0.0.5/24
- Gateway: 10.0.0.1 (your router)

- IPv6: None
- Autostart: ✅

Click **Create** and then **Apply Configuration**.

> ⚠️ Note: This will temporarily disconnect your Web UI session if done remotely.

---

## **? Step 4: Update Proxmox & Switch to Community Repos**

By default, Proxmox uses a subscription-only repo. Let’s switch it out and update everything.

1. SSH into your host or use the web terminal.
2. Add the community (no-subscription) repo:

    echo "deb http://download.proxmox.com/debian/pve bookworm pve-no-subscription" > /etc/apt/sources.list.d/pve-no-sub.list

1. Disable the enterprise repo:

    sed -i 's/^deb/#deb/' /etc/apt/sources.list.d/pve-enterprise.list

1. Update and upgrade:

    apt update && apt full-upgrade -y

1. Reboot if prompted:

    reboot

---

## **? Step 5: Run the Proxmox Helper Post-Install Script**

Save time and automate cleanup with the **Proxmox Helper Script** by tteck on GitHub.

Run this in the terminal:

    curl -sL https://github.com/tteck/Proxmox/raw/main/misc/post-pve-install.sh | bash

From the menu, you can:

- ✅ Remove enterprise repo
- ✅ Add no-subscription repo
- ✅ Disable the nag screen
- ✅ Apply common performance tweaks

Follow the prompts and let the script handle the rest.

---

## **? Step 6: Enable QEMU Guest Agent by Default**

This lets your VMs talk back to Proxmox — for things like IP display, shutdowns, and backups.

1. In the Proxmox UI, go to:**Datacenter > Options > QEMU Guest Agent**
2. Set **Default: Yes**

---

## **? Step 7: Optional – Enable Wake-on-LAN (WOL)**

If you want to power on your Proxmox box remotely:

1. Enable WOL for each NIC:

    ethtool -s enp1s0 wol g
    ethtool -s enp2s0 wol g

1. Make it persistent:

Edit /etc/network/interfaces and add:

    post-up ethtool -s enp1s0 wol g
    post-up ethtool -s enp2s0 wol g

---

## **✅ Step 8: Final Touches**

Install some handy tools:

    apt install htop nvme-cli smartmontools -y

Set up your first backup, add NFS/SMB shares, or start deploying VMs and containers.

---

## **? Summary**

You now have a clean, powerful Proxmox node built on a tiny PC:
**Component****Configuration**CPU20 cores (Intel i9 / Xeon variant)RAM96 GB DDR5Storage1TB NVMe (Proxmox OS & local-lvm)NetworkDual 2.5GbE bonded via LACPIP AddressStatic 10.0.0.5OSProxmox VE latest (no-sub repo)TweaksPost-install script, guest agent, updates ✅
---

Let me know in the comments or by DM if you’d like a follow-up post on:

- Creating a TrueNAS VM with NVMe passthrough
- Hosting VMs and LXCs on a ZFS pool
- Installing a pfSense or OPNsense firewall VM

Happy homelabbing! ??‍?

---

Would you like me to generate a diagram for this setup too?

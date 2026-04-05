---
title: Build a Proxmox Host
date_published: 2025-05-05T23:43:18.000Z
date_updated: 2025-05-05T23:43:18.000Z
tags: [series-infrastructure, proxmox, virtualization]
categories: [homelab]
---

# Building Your First Proxmox Homelab Server

*A practical guide based on real experience — including the mistakes you don’t have to make.*

---

If you want a reliable foundation for a homelab, it starts with virtualization.

Proxmox VE provides that foundation — a platform to run virtual machines, containers, and entire environments on a single piece of hardware without losing control.

This guide walks through building a Proxmox host from scratch using a Minisforum MS-01, including the decisions, tradeoffs, and mistakes I made along the way.

---

## What this setup looks like

- Single Proxmox host (Minisforum MS-01)  
- NVMe for OS and base workloads  
- Additional NVMe drives for storage and future cluster use  
- VLAN-aware networking  
- Prepared for Kubernetes and distributed workloads  

This is the baseline everything else in the homelab builds on.

---

## What You’ll Need

**Hardware:**

- A capable machine with multi-core CPU and virtualization support (VT-x/VT-d)  
- A dedicated NVMe for Proxmox + VM storage (512GB is typically sufficient)  
- At least one additional drive for data or media  
- A USB flash drive (4GB+) for the installer  
- Monitor + keyboard for initial setup  

> **On the Minisforum MS-01**  
> This mini PC is well suited for a homelab. It includes three M.2 NVMe slots, dual 10GbE SFP+, and two 2.5GbE ports.  
>  
> I used one NVMe for Proxmox, one for media storage, and one reserved for Kubernetes storage. The hardware performs well above what you would expect for its size.

---

**Software:**

- [Proxmox VE ISO](https://www.proxmox.com/en/downloads)  
- [Rufus](https://rufus.ie) (Windows) or `dd` (Linux/macOS)  

---

## Step 1: BIOS Configuration

Before installing anything, configure the BIOS correctly. Many installation issues originate here.

**Enter BIOS:** Press `Del` during boot.

### Required settings (MS-01)

**Disable Intel VMD**  
`Advanced → VMD Setup Menu → Intel VMD → Disabled`

With VMD enabled, the installer may fail to detect NVMe drives correctly.

---

**Disable ASPM (Active State Power Management)**

Disable ASPM wherever it appears in the BIOS. Power-saving features can introduce instability in a system intended to run continuously.

---

**Enable VT-d / IOMMU**  
`Advanced → CPU Configuration → Intel VT-d → Enabled`

Required for device passthrough.

---

**Set Boot Performance Mode**  
`Advanced → CPU Configuration → Boot Performance Mode → Max Non-Turbo Performance`

Turbo modes can introduce instability under virtualization workloads, particularly with hybrid CPU architectures.

---

> **Notes from experience**
> - Avoid NIC teaming unless your switch supports it properly  
> - BIOS updates may be required for high-memory configurations  
> - Stability is more important than peak performance  

---

## Step 2: Install Proxmox

1. Write the Proxmox ISO to a USB drive  
2. Boot from USB  
3. Select **Install Proxmox VE**  
4. Choose your install disk  

Proxmox will create:

- `local` — for ISOs, templates, backups  
- `local-lvm` — for VM disks  

---

**Filesystem choice**

Use `ext4` for the OS disk.

ZFS is powerful, but the additional overhead is better reserved for dedicated storage volumes rather than the host OS.

---

**Networking**

Assign a **static IP** outside your DHCP range.

---

> **Lesson learned**  
> I initially installed Proxmox on a 2TB NVMe. Most of that space was unused. A 512GB drive is typically more than enough for the OS and base storage.

---

At this point, you should be able to access the Proxmox web interface:

https://:8006

---

## Step 3: Microcode Updates (13th Gen Intel)

On newer Intel CPUs, install microcode updates immediately to prevent instability.

```bash
echo "deb http://ftp.debian.org/debian bookworm main contrib non-free-firmware" > /etc/apt/sources.list.d/non-free-firmware.list
apt update && apt install -y intel-microcode
reboot
```
Verify:
```bash
grep microcode /proc/cpuinfo | head -1
```

⸻

## Step 4: Switch to Community Repositories
```bash
echo "deb http://download.proxmox.com/debian/pve bookworm pve-no-subscription" > /etc/apt/sources.list.d/pve-no-sub.list
sed -i 's/^deb/#deb/' /etc/apt/sources.list.d/pve-enterprise.list

apt update && apt full-upgrade -y
reboot
```


⸻

## Step 5: Post-Install Script
```bash
curl -sL https://github.com/tteck/Proxmox/raw/main/misc/post-pve-install.sh | bash
```
This script handles common setup tasks and removes unnecessary friction.

⸻

## Step 6: Final Setup

Install useful tools:
```bash
apt install -y htop nvme-cli smartmontools
```
Enable QEMU Guest Agent by default:

Datacenter → Options → QEMU Guest Agent → Yes

⸻

## Step 7: Security Hardening

**SSH**

Switch to key-based authentication and disable password login.
```bash
PermitRootLogin prohibit-password
PasswordAuthentication no
```


⸻

**Firewall**

Enable the Proxmox firewall and restrict access to your local subnet.

⸻

**User management**

Create a non-root administrative account for better control and auditing.

⸻

## Step 8: Drive Health Baseline

Capture SMART data for all drives before deploying workloads.
```bash
smartctl -a /dev/nvme0
```

⸻

**Why this matters**
Establishing a baseline makes it much easier to detect degradation later.

⸻

## Step 9: Email Alerts

Configure a mail relay so the system can notify you of failures.

Most homelab setups use a Gmail relay with an app password.

⸻

## Step 10: Network Planning

Enable VLAN awareness early to avoid rework later.
```bash
bridge-vlan-aware yes
```

⸻

**Design principle**
Separate traffic logically (management, LAN, IoT, services) even if everything runs on the same hardware.

⸻

**What I’d Do Differently**
	•	Use a smaller OS disk (512GB)
	•	Use ZFS for data, not the OS
	•	Avoid NIC teaming in a homelab
	•	Plan storage and workloads before deployment

⸻

**You’re Ready**

At this point, you have a stable and hardened Proxmox host ready for workloads.

The system is:
	•	accessible
	•	monitored
	•	secured
	•	ready to scale

⸻

### What’s Next: Backups

Before deploying anything, configure backups.

**Next step**
> → [Proxmox Backup Strategy — Protecting Your Homelab with PBS](/posts/proxmox-03-backup-server)

⸻

Built on Minisforum MS-01 · Intel i9-13900H · Proxmox VE


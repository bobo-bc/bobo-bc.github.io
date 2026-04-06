---
title: Build a Proxmox Host
date_published: 2025-05-05T23:43:18.000Z
date_updated: 2025-05-05T23:43:18.000Z
tags: [series-infrastructure, proxmox, virtualization]
categories: [homelab]
---

# Building Your First Proxmox Homelab Server

*A practical guide based on real hardware, real mistakes, and deliberate tradeoffs.*

---

If you want a reliable foundation for a homelab, it starts with virtualization. Proxmox VE provides that foundation — an open source platform to run virtual machines, LXC containers, and entire environments on a single piece of hardware, with a web UI that makes it manageable without living in the terminal.

This guide walks through building a Proxmox host from scratch using a Minisforum MS-01 as the reference machine. The MS-01 is an excellent homelab platform — compact, powerful, and well priced — but some of its BIOS defaults will break your install if you don't know what to disable first. That's documented here, along with the post-install hardening that most guides skip.

---

## What this setup looks like

- Single Proxmox host on a Minisforum MS-01
- Dedicated NVMe for the OS and VM storage
- Additional NVMe drives for media and Kubernetes storage
- VLAN-aware networking ready to scale
- Hardened, monitored, and configured for long-term stability

This is the baseline everything else in the homelab builds on.

---

## What You'll Need

**Hardware:**
- A machine with multi-core CPU and virtualization support (VT-x/VT-d)
- A dedicated NVMe for Proxmox + VM storage — 512GB is typically enough
- At least one additional drive for data or media storage
- A USB flash drive (4GB+) for the installer
- Monitor + keyboard for first boot

> **On the Minisforum MS-01**
> Three M.2 NVMe slots, dual 10GbE SFP+, and two 2.5GbE ports in a compact chassis. I used one NVMe for Proxmox, one for media, and the third reserved for Kubernetes storage. It performs well above what you'd expect for its size — but read the BIOS section carefully before touching the installer.

**Software:**
- [Proxmox VE ISO](https://www.proxmox.com/en/downloads) — download the latest release
- [Rufus](https://rufus.ie) (Windows) or `dd` (Linux/macOS) to write the USB

---

## Step 1: BIOS Configuration

Get this right before touching the installer. Most failed Proxmox installs on the MS-01 trace back to BIOS settings, not the OS.

**Enter BIOS:** Press `Del` repeatedly at the Minisforum logo on boot, then navigate to Setup → Advanced.

### Disable Intel VMD

`Advanced → VMD Setup Menu → Intel VMD → Disabled`

This is the most common install-breaker on the MS-01. With VMD enabled, the Proxmox installer cannot correctly identify individual NVMe drives and will hang or fail mid-install. Disable it first.

### Disable ASPM

Search every BIOS menu for ASPM (Active State Power Management) and disable it everywhere you find it. Power-saving features designed for laptops and desktops cause random instability on a machine running continuously as a hypervisor. Hard-to-diagnose freezes are usually this.

### Enable VT-d / IOMMU

`Advanced → CPU Configuration → Intel VT-d → Enabled`

Required for device passthrough — including passing the iGPU to an LXC container for hardware-accelerated transcoding, which is covered in a later article.

### Set Boot Performance Mode

`Advanced → CPU Configuration → Boot Performance Mode → Max Non-Turbo Performance`

Turbo boost causes instability on Proxmox, particularly with the 13th gen Intel hybrid P/E-core architecture interacting with the hypervisor scheduler. This is counterintuitive but well documented in the community.

> **A few more notes from experience**
> - Don't use LACP NIC teaming unless your switch explicitly supports it. Without proper switch-side configuration it creates network instability, not speed. A single 2.5GbE port is more than sufficient for a homelab.
> - If you have 96GB RAM with a 13th gen i9, you'll need BIOS version 1.24 or later for memory stability. Contact Minisforum support to request it.

---

## Step 2: Install Proxmox

1. Write the Proxmox ISO to your USB drive using Rufus or `dd`
2. Boot from the USB (F11 at startup opens the boot menu)
3. Select **Install Proxmox VE**
4. Choose your install target drive

Proxmox automatically partitions the drive into two volumes:
- `local` — a fixed partition (~100GB) for ISOs, templates, and backups
- `local-lvm` — a thin-provisioned LVM volume using the remaining space, for VM disks

**On filesystem choice:** Use `ext4`. ZFS is powerful — checksums, self-healing, snapshots, replication — and I use it for dedicated data volumes. But the CPU and RAM overhead isn't worth it for the OS drive. A good backup strategy matters more here, and that's covered in the next article.

**On networking:** Set a static IP outside your DHCP range during install. If your router hands out `10.0.2.100–200`, use something like `10.0.2.5`. You'll be accessing the web UI at this address permanently.

**On storage sizing:** Set a strong root password and valid email address, finish the install, remove the USB, and reboot.

> **Lesson learned:** I installed Proxmox on a 2TB NVMe. Almost all of that space ended up wasted in `local-lvm` with no workloads to fill it. A 512GB drive is more than enough — the data that actually matters lives on separate storage.

Once booted, you can reach the Proxmox web UI at:

```
https://your-ip:8006
```

---

## Step 3: Microcode Updates (Critical for 13th Gen Intel)

The i9-13900H's hybrid P/E-core architecture can cause random freezes without up-to-date microcode. Do this before anything else.

Open a shell via the web UI or SSH and run:

```bash
echo "deb http://ftp.debian.org/debian bookworm main contrib non-free-firmware" > /etc/apt/sources.list.d/non-free-firmware.list
apt update && apt install -y intel-microcode
reboot
```

After rebooting, verify it applied:

```bash
grep microcode /proc/cpuinfo | head -1
```

With updated microcode and the BIOS settings from Step 1, the MS-01 runs Proxmox 24/7 without stability issues.

---

## Step 4: Switch to Community Repositories

Proxmox ships configured for its enterprise subscription repo. Without a paid subscription, updates won't install. Switch to the free community repo:

```bash
# Add the no-subscription community repo
echo "deb http://download.proxmox.com/debian/pve bookworm pve-no-subscription" > /etc/apt/sources.list.d/pve-no-sub.list

# Disable the enterprise repo
sed -i 's/^deb/#deb/' /etc/apt/sources.list.d/pve-enterprise.list

# Update and upgrade everything
apt update && apt full-upgrade -y

reboot
```

---

## Step 5: Run the Community Post-Install Script

The Proxmox Helper Scripts project automates a set of common post-install tasks with a guided menu:

```bash
curl -sL https://github.com/tteck/Proxmox/raw/main/misc/post-pve-install.sh | bash
```

From the menu, select:
- Remove enterprise repo
- Add no-subscription repo
- Disable the subscription nag screen
- Apply common performance tweaks

Follow the prompts — it's safe and actively maintained by the community.

---

## Step 6: Final Setup Touches

**Install useful host tools:**

```bash
apt install -y htop nvme-cli smartmontools
```

- `htop` — real-time CPU, RAM, and process monitor
- `nvme-cli` — NVMe health and temperature data
- `smartmontools` — disk health monitoring and alerting

**Enable QEMU Guest Agent by default:**

`Datacenter → Options → QEMU Guest Agent → Default: Yes`

This lets VMs report their IP back to Proxmox, enables graceful shutdowns from the UI, and improves backup reliability.

**Final update and reboot:**

```bash
apt update && apt full-upgrade -y && reboot
```

---

## Step 7: Security Hardening

A default Proxmox install leaves a few things open worth tightening, even on a homelab that never faces the public internet.

### Harden SSH access

Switch from password authentication to key-based login. Generate a key on your workstation if you don't have one:

```bash
ssh-keygen -t ed25519 -C "proxmox-homelab"
```

Copy it to the host:

```bash
ssh-copy-id root@10.0.2.5
```

**Verify key login works in a new terminal session first**, then disable password auth:

```bash
nano /etc/ssh/sshd_config
```

Set these values:

```
PermitRootLogin prohibit-password
PasswordAuthentication no
PubkeyAuthentication yes
```

Restart SSH:

```bash
systemctl restart sshd
```

### Enable the Proxmox firewall

`Datacenter → Firewall → Options → Firewall: Yes`

Add rules to allow only what you need from your local subnet:

`Datacenter → Firewall → Add`
- TCP port 8006 (web UI) — source: your local subnet only
- TCP port 22 (SSH) — source: your local subnet only

Then set the default input policy to DROP:

`Datacenter → Firewall → Options → Input Policy: DROP`

### Create a non-root admin user

Running everything as root works, but a named admin account gives you a cleaner audit trail and is better practice if you ever share access:

```bash
pveum user add admin@pve --password yourpassword
pveum aclmod / -user admin@pve -role Administrator
```

---

## Step 8: Drive Health Baseline

Before deploying any workloads, capture a SMART health snapshot of every drive. This gives you a reference point — drive degradation is much easier to diagnose when you know what the baseline looked like.

```bash
# List all drives
lsblk

# SMART check on each NVMe
smartctl -a /dev/nvme0
smartctl -a /dev/nvme1
smartctl -a /dev/nvme2
```

Check NVMe temperatures — particularly important on the MS-01 with all three slots populated:

```bash
nvme smart-log /dev/nvme0 | grep temperature
```

Anything consistently above 70°C under load warrants better airflow. Set up automated monitoring so you get alerts if a drive starts failing:

```bash
nano /etc/smartd.conf
```

Replace the contents with:

```
DEVICESCAN -a -o on -S on -n standby,q -s (S/../.././02|L/../../6/03) -W 4,45,55 -m root
```

Enable and start the service:

```bash
systemctl enable smartd
systemctl start smartd
```

---

## Step 9: Email Alerts

Proxmox will notify you about drive failures, backup errors, and system issues — but only if you configure a mail relay. Most homelabbers use a Gmail relay with an app password.

**Install postfix:**

```bash
apt install -y postfix libsasl2-modules mailutils
```

Choose **Internet Site** when prompted and set the mail name to your hostname.

**Configure Gmail relay** — add to `/etc/postfix/main.cf`:

```
relayhost = [smtp.gmail.com]:587
smtp_use_tls = yes
smtp_sasl_auth_enable = yes
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_sasl_security_options = noanonymous
smtp_tls_CAfile = /etc/ssl/certs/ca-certificates.crt
```

**Create the credentials file** at `/etc/postfix/sasl_passwd`:

```
[smtp.gmail.com]:587 you@gmail.com:your-app-password
```

> To generate a Gmail app password: Google Account → Security → 2-Step Verification → App passwords.

**Apply the configuration:**

```bash
chmod 600 /etc/postfix/sasl_passwd
postmap /etc/postfix/sasl_passwd
systemctl restart postfix
```

**Test it:**

```bash
echo "Test from Proxmox" | mail -s "Proxmox alert test" you@gmail.com
```

Then set the notification address in the UI: `Datacenter → Options → Email from address`.

---

## Step 10: Network Planning and VLANs

VLANs let you logically separate traffic on the same physical network without extra cabling. Even if you don't use them immediately, enabling VLAN awareness on your bridge now avoids having to rebuild networking later.

**Enable VLAN awareness on vmbr0:**

```bash
nano /etc/network/interfaces
```

Add `bridge-vlan-aware yes` to your bridge definition:

```
auto vmbr0
iface vmbr0 inet static
    address 10.0.2.5/24
    gateway 10.0.2.1
    bridge-ports enp0s31f6
    bridge-stp off
    bridge-fd 0
    bridge-vlan-aware yes
```

Apply without rebooting:

```bash
ifreload -a
```

**A sensible VLAN layout to grow into:**

| VLAN | Purpose                                         |
| ---- | ----------------------------------------------- |
| 1    | Management (Proxmox host, router)               |
| 2    | Trusted LAN (workstations, laptops)             |
| 3    | Media (Plex, NAS)                               |
| 10   | IoT (isolated — these are often poorly secured) |

**On the MS-01's four NICs — a practical layout:**

| NIC         | Type   | Suggested use                   |
| ----------- | ------ | ------------------------------- |
| i226-V      | 2.5GbE | Proxmox management + LAN bridge |
| i226-LM     | 2.5GbE | Spare / VM-only traffic         |
| SFP+ port 1 | 10GbE  | NAS / high-speed storage        |
| SFP+ port 2 | 10GbE  | Future cluster or node link     |

---

## Step 11: CPU Core Pinning (13th Gen i9)

***Note**:Do this step if you are tight on resources and really want to optimize otherwise let the host manage itself*

The i9-13900H has 6 Performance cores (P-cores, CPUs 0–11 with hyperthreading) and 8 Efficiency cores (E-cores, CPUs 12–19). By default, Proxmox's scheduler treats them identically — which means a heavy VM might land on E-cores while lightweight containers waste P-core capacity.

Verify your layout first:

```bash
lscpu -e
```

For a mixed workload environment — Kubernetes workers, databases, Plex, and background services — a sensible pinning strategy is:

```bash
# Kubernetes workers — split P-cores by workload type
qm set <worker-ml-id>  --affinity 0-5    # Immich ML — highest CPU demand
qm set <worker-db-id>  --affinity 4-9    # Postgres/Redis — latency sensitive
qm set <worker-gen-id> --affinity 6-11   # General pods

# Lightweight services — float across E-cores, CPU limit as a ceiling
qm set <ha-id>   --affinity 12-19 --cpulimit 2
pct set <plex-id> --affinity 10-19 --cpulimit 4
pct set <pbs-id>  --affinity 12-19 --cpulimit 4

# Control plane — just set a ceiling, let it float
qm set <cp1-id> --cpulimit 2
qm set <cp2-id> --cpulimit 2
qm set <cp3-id> --cpulimit 2
```

Don't pin the Proxmox host itself — the hypervisor should float freely. Pinning is only for VMs and LXCs.

---

## What I'd Do Differently

**Storage sizing:** A 512GB NVMe for Proxmox is sufficient. The extra capacity in a larger drive goes to waste in `local-lvm` unless you have a specific plan for it. Data that matters lives on dedicated storage.

**ZFS for data, not the OS:** ZFS is genuinely excellent for data volumes — checksums, self-healing, snapshots, replication. But the CPU and RAM overhead doesn't justify its use for the OS drive.

**Skip NIC teaming:** LACP bonding requires proper switch support. Without it you get instability, not redundancy. A single 2.5GbE connection is plenty for a homelab.

**Plan before you provision:** Know roughly what services you want to run, how much disk each needs, and whether they'll share storage or get dedicated volumes. Retrofitting storage decisions is painful.

---

## You're Ready

At this point you have a stable, hardened Proxmox host ready for workloads. SSH is locked down, the firewall is in place, drives are monitored, alerts are configured, and the network is ready to scale.

---

## What's Next: Backups

Before you deploy your first VM or LXC, set up backups. Proxmox Backup Server (PBS) supports incremental backups, deduplication, and encryption — and integrates directly with the Proxmox UI. It takes less than an hour to configure and will save you enormously when something goes wrong.

**→ Read the next article: [Homelab Backup Strategy — Protecting Your Proxmox Host with PBS](/posts/proxmox-03-backup-server)**






---
title: Build a Proxmox backup server
layout: homelab
series: infrastructure
order: 2
tags: [series-infrastructure, proxmox, virtualization]
categories: [homelab]
---
# Homelab Backup Strategy: Protecting Your Proxmox Host with PBS

*A two-tier backup solution using Proxmox Backup Server — fast local recovery and off-host resilience on a Raspberry Pi.*

---

When people think about homelab reliability, they usually jump straight to clustering and high availability. But for most homelabbers, that's solving the wrong problem at significant cost and complexity. What you actually need is a clear-headed answer to three distinct questions:

**Availability** — can your services be accessed right now? Redundancy and clustering address this, but come with real hardware and maintenance costs.

**Resilience** — can you recover quickly from a disruption like a failed VM or corrupted container? Good state management and fast local backups address this.

**Recoverability** — can you rebuild from a catastrophic failure like a dead host or lost storage? This is where a proper backup strategy earns its keep.

For my homelab, I made a deliberate choice: skip the expense and complexity of a Proxmox cluster and instead invest in a robust two-tier backup solution that gives me fast local recovery *and* off-host resilience for catastrophic scenarios. This article documents exactly how I built it.

---

## The Architecture

The cornerstone of the solution is **Proxmox Backup Server (PBS)** — an open source backup solution designed specifically for Proxmox environments. It supports incremental backups, deduplication, encryption, and has deep integration with the Proxmox UI.

I run two PBS instances:

**Tier 1 — PBS LXC (primary, fast)**
A PBS container running inside Proxmox itself, backed by a dedicated USB M.2 SATA drive. This is the workhorse — used for day-to-day restores when a VM or LXC has problems. Because it's local to the host, restores are fast.

**Tier 2 — Raspberry Pi 4 PBS (off-host, resilient)**
A Raspberry Pi 4 running PBS natively, also backed by a dedicated USB M.2 SATA drive. The Pi acts as a sync target for the LXC PBS, meaning it holds a complete copy of all backups. If the Proxmox host itself fails, the Pi has everything needed to rebuild — the host OS backup, the PBS LXC image, and all VM/LXC backups.

The Pi is intentionally not the primary backup target. Its ARM CPU makes it too slow for high-frequency backup jobs. But as a sync destination it's perfect — low power, always on, completely independent of the main host.

```
┌─────────────────────────────────┐
│         Proxmox Host            │
│                                 │
│  ┌──────────┐  ┌─────────────┐  │
│  │  VMs &   │  │  PBS LXC    │  │
│  │  LXCs    │─▶│  (Primary)  │  │
│  └──────────┘  └──────┬──────┘  │
│                        │ sync   │
└────────────────────────┼────────┘
                         │
                         ▼
              ┌─────────────────┐
              │  Raspberry Pi 4 │
              │  PBS (Off-host) │
              └─────────────────┘
```

---

## Part 1: Building the PBS LXC

### Install via Proxmox Helper Scripts

The easiest way to get PBS running as an LXC is via the community helper scripts. In your Proxmox shell:

Navigate to the [Proxmox Helper Scripts community site](https://community-scripts.org/scripts/proxmox-backup-server), copy the installer command and paste it into the **Proxmox host shell** (not an LXC console):

```bash
# The command will look something like this — always copy the current version from the site
bash -c "$(wget -qLO - https://github.com/community-scripts/ProxmoxVE/raw/main/ct/proxmox-backup-server.sh)"
```

During setup, when prompted:
- Assign a **static IP outside your DHCP scope** — e.g. if your router hands out `10.0.2.100–200`, use something like `10.0.2.15`
- Give it a memorable hostname like `pbs`
- Allocate at least 2GB RAM and 2 cores — PBS is efficient but deduplication is CPU-bound

### Mount your backup drive

Once the LXC is created, you need to attach the USB M.2 SATA drive as a mount point so PBS has somewhere to store backups.

First, identify the drive on the Proxmox host:

```bash
lsblk
# Look for your USB drive — typically /dev/sda or /dev/sdb
fdisk -l /dev/sda
```

Format it if it's new:

```bash
mkfs.ext4 /dev/sda
```

Create a mount point directory on the host:

```bash
mkdir -p /mnt/pbs-storage
```

Add it to `/etc/fstab` for persistence:

```bash
echo "/dev/sda /mnt/pbs-storage ext4 defaults 0 2" >> /etc/fstab
mount -a
```

Now attach it to the LXC via the Proxmox UI:

`LXC 103 → Resources → Add → Mount Point`
- Path on host: `/mnt/pbs-storage`
- Path in container: `/mnt/backup`
- Set appropriate size limit if desired

Reboot the LXC:

```bash
pct reboot <lxc-id>
```

### Access the PBS web UI

Open a browser and navigate to:

```
https://10.0.2.15:8007
```

Log in with `root` and the password you set during install. You'll see the PBS dashboard.

### Create a Datastore

A datastore is where PBS stores backup chunks. Think of it as the backup library.

`Administration → Datastore → Create`
- Name: `main` (or whatever you prefer)
- Backing Path: `/mnt/backup`
- Enable deduplication: ✅ (this is a major PBS advantage — identical blocks across backups are stored only once)
- Enable garbage collection: ✅

Once created, PBS will initialize the datastore structure on your drive.

### Set up a backup user and token

Rather than using root credentials in Proxmox, create a dedicated backup user:

In PBS: `Configuration → Access Control → Users → Add`
- Username: `backup-user`
- Realm: `pbs`

Then create an API token for that user (tokens are safer than passwords for automated jobs):

`Configuration → Access Control → API Tokens → Add`
- User: `backup-user@pbs`
- Token name: `proxmox`
- Privilege Separation: unchecked (inherits user permissions)

Copy the token secret — you'll only see it once.

Now grant the user `DatastoreBackup` privilege on your datastore:

`Administration → Datastore → main → Permissions → Add`
- User: `backup-user@pbs`
- Role: `DatastoreBackup`

---

## Part 2: Connect PBS to Proxmox

### Add PBS as a storage target

In the **Proxmox web UI**:

`Datacenter → Storage → Add → Proxmox Backup Server`

- ID: `pbs-main`
- Server: `10.0.2.15`
- Username: `backup-user@pbs`
- Password/Token: paste your API token in format `backup-user@pbs!proxmox=<token-secret>`
- Datastore: `main`
- Fingerprint: you can get this from PBS under `Configuration → Certificates`

Click Add — Proxmox will verify the connection.

### Configure a backup schedule

`Datacenter → Backup → Add`

- Storage: `pbs-main`
- Schedule: choose a time that works for your usage pattern — nightly during off-hours is a sensible default for most homelabs
- Mode: **Snapshot** is fastest (no downtime), **Suspend** is safe for consistency, **Stop** is most consistent but causes downtime
- Retention: configure how many backups to keep per VM

For retention strategy, PBS documentation covers the `keep-last`, `keep-daily`, `keep-weekly` and `keep-monthly` options well. A reasonable homelab starting point is keep-last=3, keep-daily=7, keep-weekly=4.

See the [PBS documentation on backup retention](https://pbs.proxmox.com/docs/backup-client.html#backup-pruning) for full details.

---

## Part 3: Building the Raspberry Pi PBS

### What you'll need

- Raspberry Pi 4 (4GB RAM minimum recommended)
- MicroSD card or small USB flash drive for the OS (8GB+)
- USB M.2 SATA drive for backup storage
- Ethernet cable — hardwire it, don't rely on WiFi for backup traffic

### Install PBS on the Pi

The [PIPBS project on GitHub](https://github.com/dexogen/pipbs) provides a purpose-built installer that compiles and configures PBS for ARM (the Pi's architecture). Official PBS binaries are x86-only, so this project fills that gap.

Follow the installation instructions in the PIPBS repository README — the project is well documented and handles the ARM compilation automatically.

Once installed, assign the Pi a static IP on your network. Either configure it on the Pi itself:

```bash
# Edit the network config
sudo nano /etc/dhcpcd.conf

# Add at the bottom
interface eth0
static ip_address=10.0.2.20/24
static routers=10.0.2.1
static domain_name_servers=10.0.2.1
```

Or reserve the IP in your router's DHCP settings by MAC address — either approach works.

### Create a datastore on the Pi

Mount and format your USB M.2 SATA drive:

```bash
# Identify the drive
lsblk

# Format (adjust device name as needed)
sudo mkfs.ext4 /dev/sda

# Create mount point and mount
sudo mkdir -p /mnt/pbs-storage
echo "/dev/sda /mnt/pbs-storage ext4 defaults 0 2" | sudo tee -a /etc/fstab
sudo mount -a
```

Access the Pi PBS web UI:

```
https://10.0.2.20:8007
```

Create a datastore pointing at `/mnt/pbs-storage` following the same steps as Part 1.

Create a sync user on the Pi PBS with `DatastoreBackup` and `DatastoreAudit` privileges — the LXC PBS will authenticate as this user when pushing sync jobs.

---

## Part 4: Configuring Sync from LXC PBS to Pi PBS

This is the glue that makes the two-tier architecture work. The LXC PBS will push a copy of all backups to the Pi PBS on a schedule.

### Add the Pi as a remote in LXC PBS

In the **LXC PBS web UI** (`https://10.0.2.15:8007`):

`Configuration → Remotes → Add`
- Name: `pi-pbs`
- Host: `10.0.2.20`
- Port: `8007`
- Username: `sync-user@pbs` (the user you created on the Pi)
- Password: the sync user's password
- Fingerprint: get this from the Pi PBS under `Configuration → Certificates`

Click Test Connection to verify.

### Create a sync job

`Administration → Sync Jobs → Add`
- Remote: `pi-pbs`
- Remote Store: the datastore name on the Pi
- Local Store: `main` (your LXC PBS datastore)
- Schedule: daily, offset a few hours after your main backup job completes — e.g. if backups run at 2am, sync at 5am
- Remove Vanished: ✅ — keeps the Pi in sync when old backups are pruned on the primary
- Sync direction: **Push** (LXC PBS → Pi PBS)

Save the sync job. You can trigger it manually the first time to verify it works:

`Sync Jobs → select job → Run Now`

The first sync will take a while depending on how much data you have. Subsequent syncs are incremental thanks to PBS deduplication — only changed chunks are transferred.

### Verify the sync

Once the first sync completes, log into the Pi PBS UI and confirm the datastores shows the same backup snapshots as the primary:

`Administration → Datastore → main → Content`

You should see matching VM and LXC backup entries.

---

## Restoring from Backup

### Scenario 1: A VM or LXC needs restoring (host is healthy)

Use the fast local PBS LXC. In Proxmox UI:

`Datacenter → Storage → pbs-main → select snapshot → Restore`

Choose the target node and storage, then restore. For LXCs this typically takes seconds to minutes. VMs depend on disk size.

### Scenario 2: The Proxmox host itself has failed

This is where the Pi earns its place. You'll need to:

1. Reinstall Proxmox on the host (refer to the [Proxmox build guide](#))
2. Reinstall the PBS LXC using the helper script
3. Add the Pi PBS as a remote in the new LXC PBS
4. Sync backups back from the Pi to the LXC PBS
5. Restore VMs and LXCs from the recovered backup data

It's worth doing a **dry run of this scenario** before you actually need it. Restore a non-critical VM to a test environment to confirm your backups are good and you understand the recovery process.

---

## Lessons Learned

**Test your restores.** A backup you've never restored from is a backup you can't trust. Schedule a quarterly restore test of at least one VM or LXC.

**Hardwire the Pi.** WiFi sync is unreliable for large backup transfers. A dedicated ethernet run to your switch is worth the effort.

**Offset your schedules.** Don't run backups and sync jobs at the same time. Stagger them so the sync job starts after backups complete — overlapping jobs compete for I/O and slow everything down.

**Label your drives.** USB drives look identical. Label them clearly — a mix-up during a stressful recovery is the last thing you want.

**The Pi is slow for a reason.** Don't be tempted to use the Pi as the primary backup target to save money. Its ARM CPU makes deduplication and indexing slow under load. It excels as a sync target because those jobs run infrequently and off-peak.

---

## Summary

| Tier    | Location       | Purpose               | Speed                |
| ------- | -------------- | --------------------- | -------------------- |
| PBS LXC | Inside Proxmox | Day-to-day restores   | Fast                 |
| Pi PBS  | Off-host       | Catastrophic recovery | Slow but independent |

Two PBS instances, one sync job, and you have a homelab backup solution that would cost thousands in enterprise hardware replicated for the price of a Raspberry Pi and a couple of M.2 drives.


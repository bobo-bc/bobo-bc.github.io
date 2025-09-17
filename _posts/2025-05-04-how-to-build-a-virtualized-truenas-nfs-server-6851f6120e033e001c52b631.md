---
title: A virtualized TrueNas NFS server in Proxmox
slug: how-to-build-a-virtualized-truenas-nfs-server-6851f6120e033e001c52b631
date_published: 2025-05-04T23:52:08.000Z
date_updated: 2025-06-17T23:51:52.000Z
tags: #Migrated-1750201873783, #wp, #wp-post, #Import 2025-06-17 16:11, Homelab
---

**Published by Bobo on May 4, 2025**

In this guide, I’ll walk you through setting up **TrueNAS** as a virtual machine in **Proxmox VE**, with **two 2TB NVMe SSDs** passed through to create a **mirrored ZFS pool**. You’ll also configure a static IP and set up an **NFS share** to serve files across your network—perfect for a homelab or Proxmox storage backend.

---

## **What You’ll Need**

- A working **Proxmox VE** host
- **Two 2TB NVMe SSDs** (not already used by Proxmox)
- Latest **TrueNAS ISO** → [Download here](https://www.truenas.com/download)
- Static IP: 10.0.0.3
- (Optional) SSH access to your Proxmox host

---

## **1. Upload the TrueNAS ISO to Proxmox**

1. In the Proxmox web UI, click your node.
2. Navigate to **local > ISO Images**.
3. Click **Upload**, then select the downloaded TrueNAS ISO.

---

## **2. Create the TrueNAS Virtual Machine**

- Click **Create VM**.

In the **General** tab:

- Name: truenas

In the **OS** tab:

- Select the TrueNAS ISO
- Guest OS Type: Linux

In the **System** tab:

- BIOS: OVMF (UEFI)
- Machine: q35
- Enable **QEMU Guest Agent**

In the **Hard Disk** tab:

- Bus/Device: SCSI
- Storage: local
- Size: 16 GB (for OS only)

In the **CPU** tab:

- Cores: 3
- Type: host

In the **Memory** tab:

- Allocate 14336 MB (~14 GB)

In the **Network** tab:

- Bridge: vmbr0
- Model: VirtIO (paravirtualized)

Click **Finish**, but **don’t start the VM yet**.

---

## **3. Pass Through the NVMe Drives**

Before proceeding, **shut down the VM** if it’s running:

- Go to **VM > Summary**
- Click **Shutdown** (or **Stop**, if unresponsive)

### **Attach NVMe Drives as Raw Disks**

1. Go to **VM > Hardware**
2. Click **Add > Hard Disk**
3. In the popup:
- **Bus/Device**: VirtIO (recommended)
- **Storage**: Select **“Do not use any storage”**
- **Disk image**: Leave empty
- Enable **“Use Physical Disk”**
- Enter device path manually, e.g. /dev/nvme0n1

4. Repeat for /dev/nvme1n1

> You can verify NVMe drive paths with lsblk or fdisk -l from the Proxmox shell.

### **Confirm Disks Are Attached**

Go back to **VM > Hardware** and verify you see:

- VirtIO Disk 2 → /dev/nvme0n1
- VirtIO Disk 3 → /dev/nvme1n1

---

## **4. Install TrueNAS**

1. Start the VM
2. Open the **Console** tab
3. Run through the TrueNAS installer
- Install onto the 16GB virtual disk

4. Reboot when done

**After reboot**, remove the ISO:

- Go to **VM > Hardware**
- Right-click CD-ROM → **Remove**

---

## **5. Set a Static IP for TrueNAS**

1. From the VM console, select **Configure Network Interfaces**
2. Assign:
- IP: 10.0.0.3
- Netmask: 24
- Gateway: 10.0.0.1 (your router?)
- DNS: 1.1.1.1 or similar

Now open a browser and go to **http://10.0.0.3** to access the web UI.

---

## **6. Create a Mirrored ZFS Pool**

1. In the web UI, go to **Storage > Pools**
2. Click **Add > Create new pool**
3. Name: nvme_pool
4. Select both NVMe disks
5. Layout: **Mirror** (adds redundancy)
6. Click **Create**

> This will wipe all data on the drives!

---

## **7. Set Up an NFS Share**

### **Create a Dataset**

1. Navigate to:**Storage > Pools > nvme_pool > Add Dataset**
2. Name it nfs

### **Configure NFS Sharing**

1. Go to **Sharing > Unix (NFS) Shares**
2. Click **Add**:
- Path: /mnt/nvme_pool/nfs
- Check: **All dirs**
- Allowed networks: 10.0.0.0/24
- Maproot User: root
- Maproot Group: wheel

### **Enable NFS Service**

- Go to **Services**
- Start **NFS**
- Enable **Start Automatically**

---

## **8. Test the NFS Share**

On a Linux client or another Proxmox host, run:

    sudo mount -t nfs 10.0.0.3:/mnt/nvme_pool/nfs /mnt/test

If it mounts successfully, congrats—you’ve got a working NFS server powered by virtualized TrueNAS!

---

## **Summary**

You now have:

- A **TrueNAS VM** running in **Proxmox VE**
- **3 vCPUs**, **14GB RAM**, and **16GB** OS disk
- Two **2TB NVMe SSDs** passed through as raw disks
- **ZFS mirrored pool** for redundancy
- **Static IP**: 10.0.0.3
- **NFS share** available to your LAN

This setup is ideal for:

- Hosting Proxmox VM backups
- Serving files to your home network
- Running iSCSI or SMB shares in the future
- Experimenting with ZFS snapshots and replication

---

Let me know if you’d like help turning this into a downloadable PDF, adding screenshots, or creating a diagram for the architecture!

---
layout: post
title: "GlusterFS"
date: 2025-09-13 11:40:00 +0800
categories: [swarm]
tags: [Storage]
---

##Install and Configure GlusterFS for Distributed Storage

Install GlusterFS on All Nodes

Run the following command on each master node:

```bash
sudo apt update && sudo apt install -y glusterfs-server
sudo systemctl start glusterd
sudo systemctl enable glusterd
```

### Prepare Storage Disks

On each node, format and mount the storage disk:

```bash
sudo mkfs.xfs -f /dev/sdb
sudo mkdir -p /data/glusterfs/myvol1/brick1
echo "/dev/sdb /data/glusterfs/myvol1/brick1 xfs defaults 0 0" | sudo tee -a /etc/fstab
sudo mount -a
```

### Initialize the GlusterFS Cluster

On Node1, probe the other nodes:

```bash
gluster peer probe 10.0.0.22
gluster peer probe 10.0.0.23
gluster pool list
```

### Create a Distributed Volume

On Node1, create the volume:

```bash
gluster volume create dockervol1 disperse 3 redundancy 1 \
10.0.0.21:/data/glusterfs/myvol1/brick1 \
10.0.0.22:/data/glusterfs/myvol1/brick1 \
10.0.0.23:/data/glusterfs/myvol1/brick1
```

Start the volume:

```bash
gluster volume start dockervol1
```

Mount the Volume on each Nodes

```bash
sudo mkdir /mnt/docker-storage
echo "localhost:/dockervol1 /mnt/docker-storage glusterfs defaults,_netdev 0 0" | sudo tee -a /etc/fstab
sudo mount -a
```

Verify the volume:

```bash
gluster volume info dockervol1
```

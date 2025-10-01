---
layout: post
title: "01- A Three Node Master Swarm"
date: 2025-09-29 11:40:00 +0800
categories: [swarm]
tags: [build]
---

## Introduction
Ensuring high availability in a Docker Swarm cluster is crucial for maintaining uptime and preventing service disruptions. In this guide, we will configure Keepalived for virtual IP failover across a three-master node Debian OS cluster and GlusterFS for distributed storage for config files and databases.
Will use NFS as shared storage for media and other shared files. 

### Node Assignments:

- Node1 (Primary Master): 10.0.0.21
- Node2 (Backup Master): 10.0.0.22
- Node3 (Backup Master): 10.0.0.23
- Virtual IP (VIP): 10.0.0.20

***

## Step 1: Install Docker Swarm on All Nodes

### Ensure Docker is installed on all nodes:

```bash
sudo apt update && sudo apt install -y docker.io
sudo systemctl enable docker
sudo systemctl start docker
```

### Initialize Docker Swarm on Node1:

```bash
docker swarm init --advertise-addr 10.0.0.20
```

Retrieve the join token for manager nodes:

```bash
docker swarm join-token manager
```
On Node2 and Node3, join the Swarm using the token:

```bash
docker swarm join --token <TOKEN> --advertise-addr <NODE_IP>
```

Verify the cluster:

```bash
docker node ls
```
***

# Network

 If you are configured using 192.x.x.x. or 172.x.x.x then skip ahead. If your homelab network is 10.x.x.x, you will need to do the following.

Bug: The overlay ingress network that is automatically created was in the same range as the nodes if you are using 10.x.x.x network. 
Our Nodes have IPs from 10.0.0.x and the network was set
You can check that with commands:

```bash
docker network ls
```

| NETWORK ID   | NAME                  | DRIVER  | SCOPE |
|--------------|-----------------------|:-------:|:-----:|
| kpiocqticjlx | ingress               | overlay | swarm |
| k55h53e1e97d | portainer_agent_network | overlay | swarm |

using the Network ID of the ingress network

```bash
docker network inspect --format='{{json .IPAM.Config}}' kpiocqticjlx
```
If the range is the same as your node IPs, you need to change it.

```bash
docker network rm ingress
```

now create a new ingress defining a different ip range

```bash
docker network create --driver overlay --ingress --subnet 172.16.0.0/16 --gateway 172.16.0.1 ingress
```

Restart all your nodes.

***

# Step 2: Install and Configure Keepalived for VIP Failover

KeepAlived sets a primary, secondary, and so-on of hosts that resonds to a shared IP address. When Primary does not respond the secondary automatically takes over. 

## Install Keepalived on All Nodes

Run the following command on each master node:

```bash
sudo apt update && sudo apt install -y keepalived
```
Configure Keepalived on Node1 (10.0.0.21)

```bash
sudo nano /etc/keepalived/keepalived.conf
```
Add the following configuration:

```bash
cfg
vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 51
    priority 200
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass keepalivedpassword
    }
    virtual_ipaddress {
        10.0.0.20/24
    }
}
```

Restart Keepalived:

```bash
sudo systemctl restart keepalived
sudo systemctl enable keepalived
```

### Configure Keepalived on Node2 (10.0.0.22)

```bash
sudo nano /etc/keepalived/keepalived.conf
```

Modify the configuration:

```bash
cfg
vrrp_instance VI_1 {
    state BACKUP
    interface eth0
    virtual_router_id 51
    priority 150
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass keepalivedpassword
    }
    virtual_ipaddress {
        10.0.0.20/24
    }
}
```

Restart Keepalived:

```bash
sudo systemctl restart keepalived
sudo systemctl enable keepalived
```

### Configure Keepalived on Node3 (10.0.0.23)

```bash
sudo nano /etc/keepalived/keepalived.conf
```

Modify the configuration:

```bash
cfg
vrrp_instance VI_1 {
    state BACKUP
    interface eth0
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass keepalivedpassword
    }
    virtual_ipaddress {
        10.0.0.20/24
    }
}
```

Restart Keepalived:

```bash
sudo systemctl restart keepalived
sudo systemctl enable keepalived
```

### Verify VIP Failover
On Node1, check if the VIP is assigned:

```bash
ip a show eth0
```

Stop Keepalived on Node1:

```bash
sudo systemctl stop keepalived
```

Check Node2:

```bash
ip a show eth0
```
The VIP should now be assigned to Node2, confirming failover works.

***

## Step 3: Install and Configure GlusterFS for Distributed Storage

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

*** 

## Step 4: Ensure Docker Swarm Uses VIP for Connectivity

Now that Keepalived is handling the VIP (10.0.0.20), update Docker Swarm manager initialization to reference the VIP instead of individual master nodes:

```bash
docker swarm init --advertise-addr 10.0.0.20
```

For additional worker nodes, use:

```bash
docker swarm join --token <TOKEN> --advertise-addr <WORKER_IP>
```

This ensures Swarm joins are processed by the VIP, keeping the cluster highly available.

### Conclusion
With GlusterFS and Keepalived, your three-master node Debian OS cluster now has automatic failover for the control plane and distributed storage for persistent data. The VIP (10.0.0.20) dynamically moves between nodes based on availability, ensuring high availability and resiliency in case of failure.

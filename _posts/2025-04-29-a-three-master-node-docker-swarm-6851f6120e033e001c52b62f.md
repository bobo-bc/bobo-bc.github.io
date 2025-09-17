---
title: A three master node docker swarm
slug: a-three-master-node-docker-swarm-6851f6120e033e001c52b62f
date_published: 2025-04-30T02:15:08.000Z
date_updated: 2025-04-30T02:15:08.000Z
tags: Docker Swarm, #Migrated-1750201873783, #wp, #wp-post, #Import 2025-06-17 16:11
---

## **ntroduction**

Ensuring **high availability** in a **Docker Swarm cluster** is crucial for maintaining uptime and preventing service disruptions. In this guide, we will configure **GlusterFS** for distributed storage and **Keepalived** for virtual IP failover across a **three-master node Debian OS cluster**.

### **Node Assignments**

- **Node1 (Primary Master):**`10.0.0.21`
- **Node2 (Backup Master):**`10.0.0.22`
- **Node3 (Backup Master):**`10.0.0.23`
- **Virtual IP (VIP):**`10.0.0.20`

## **Step 1: Install Docker Swarm on All Nodes**

Ensure Docker is installed on all nodes:

    sudo apt update && sudo apt install -y docker.io
    sudo systemctl enable docker
    sudo systemctl start docker
    

Initialize Docker Swarm on **Node1**:

    docker swarm init --advertise-addr 10.0.0.20
    

Retrieve the **join token** for manager nodes:

    docker swarm join-token manager
    

On **Node2** and **Node3**, join the Swarm using the token:

    docker swarm join --token <TOKEN> --advertise-addr <NODE_IP>
    

Verify the cluster:

    docker node ls
    

## **Step 2: Install and Configure Keepalived for VIP Failover**

### **Install Keepalived on All Nodes**

Run the following command on **each master node**:

    sudo apt update && sudo apt install -y keepalived
    

### **Configure Keepalived on Node1 (**`10.0.0.21`**)**

    sudo nano /etc/keepalived/keepalived.conf
    

Add the following configuration:

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
    

Restart Keepalived:

    sudo systemctl restart keepalived
    sudo systemctl enable keepalived
    

### **Configure Keepalived on Node2 (**`10.0.0.22`**)**

    sudo nano /etc/keepalived/keepalived.conf
    

Modify the configuration:

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
    

Restart Keepalived:

    sudo systemctl restart keepalived
    sudo systemctl enable keepalived
    

### **Configure Keepalived on Node3 (**`10.0.0.23`**)**

    sudo nano /etc/keepalived/keepalived.conf
    

Modify the configuration:

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
    

Restart Keepalived:

    sudo systemctl restart keepalived
    sudo systemctl enable keepalived
    

### **Verify VIP Failover**

On **Node1**, check if the VIP is assigned:

    ip a show eth0
    

Stop Keepalived on **Node1**:

    sudo systemctl stop keepalived
    

Check **Node2**:

    ip a show eth0
    

The VIP should now be assigned to **Node2**, confirming failover works.

## **Step 3: Install and Configure GlusterFS for Distributed Storage**

### **Install GlusterFS on All Nodes**

Run the following command on **each master node**:

    sudo apt update && sudo apt install -y glusterfs-server
    sudo systemctl start glusterd
    sudo systemctl enable glusterd
    

### **Prepare Storage Disks**

On **each node**, format and mount the storage disk:

    sudo mkfs.xfs -f /dev/sdb
    sudo mkdir -p /data/glusterfs/myvol1/brick1
    echo "/dev/sdb /data/glusterfs/myvol1/brick1 xfs defaults 0 0" | sudo tee -a /etc/fstab
    sudo mount -a
    

### **Initialize the GlusterFS Cluster**

On **Node1**, probe the other nodes:

    gluster peer probe 10.0.0.22
    gluster peer probe 10.0.0.23
    gluster pool list
    

### **Create a Distributed Volume**

On **Node1**, create the volume:

    gluster volume create dockervol1 disperse 3 redundancy 1 \
    10.0.0.21:/data/glusterfs/myvol1/brick1 \
    10.0.0.22:/data/glusterfs/myvol1/brick1 \
    10.0.0.23:/data/glusterfs/myvol1/brick1
    

Start the volume:

    gluster volume start dockervol1
    

### **Mount the Volume on All Nodes**

On **each node**, mount the volume:

    sudo mkdir /mnt/docker-storage
    echo "localhost:/dockervol1 /mnt/docker-storage glusterfs defaults,_netdev 0 0" | sudo tee -a /etc/fstab
    sudo mount -a
    

Verify the volume:

    gluster volume info dockervol1
    

## **Step 4: Ensure Docker Swarm Uses VIP for Connectivity**

Now that Keepalived is handling the VIP (`10.0.0.20`), update **Docker Swarm manager initialization** to reference the VIP instead of individual master nodes:

    docker swarm init --advertise-addr 10.0.0.20
    

For additional worker nodes, use:

    docker swarm join --token <TOKEN> --advertise-addr <WORKER_IP>
    

This ensures **Swarm joins are processed by the VIP**, keeping the cluster **highly available**.

## **Conclusion**

With **GlusterFS** and **Keepalived**, your **three-master node Debian OS cluster** now has **automatic failover** for the control plane and **distributed storage** for persistent data. The VIP (`10.0.0.20`) dynamically moves between nodes based on availability, ensuring **high availability and resiliency** in case of failure.

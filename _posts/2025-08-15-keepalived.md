---
layout: post
title: "KeepAlived Virtual IP"
date: 2025-09-27 15:30:00 +0800
categories: [swarm]
tags: [build]
---

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

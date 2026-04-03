---
layout: post
title: "Swarm Nodes from vm template"
date: 2025-09-29 10:30:00 +0800
categories: [swarm]
---

## The Build
To make life easier and provide a consistent experience we are going to create a proxmox vm template. In this exercise we will use Dietpi which is a minimal Debian OS. If you are building on multiple raspberry pi's you can skip this section. 

## Create a VM template
Create a dietpi virtual machine by using this install script run from the proxmox pve shell that was originally published here: 

[dazeb install script](https://github.com/dazeb/proxmox-dietpi-installer)

```bash
bash <(curl -sSfL https://raw.githubusercontent.com/dazeb/proxmox-dietpi-installer/main/dietpi-install.sh)
```

Things to note:
- 8 Gig ram
- 2 core CPU and set the the cpu driver type to "x86-64-v2"
- 16 Gig for Os drive on local proxmox storage.

```bash
DietPi-Config
```

- set timezone
- name call this one "node"
- fix IP to something not needed like 10.0.0.19

```bash
DietPi-Software
```

- select both docker and docker compose.
- select install and dietpi will install components and update the OS.
- reboot

Lastly add in proxmox for the vms hardware

- 50 gig Hard Disk (iscsi) on "local" proxmox storage. - to be used for glusterfS later.
- Then shutdown the VM

## create template

Right click the vm and convert to template.

The nodes can be created from cloning the template.

Create the nodes

Right click the template and clone.

- set the name for the node (node1)
- set the VM ID to a high numberer number so all the swarm nodes can be sequential like 150
- set mode to Full clone

After the node has been created log in and enter

```bash
DietPi-Config
```

Configure each clone by changing name and fixing IP:

| Node  | IP Address |
|-------|------------|
| node1 | 10.0.0.21  |
| node2 | 10.0.0.22  |
| node3 | 10.0.0.23  |


One final reboot.

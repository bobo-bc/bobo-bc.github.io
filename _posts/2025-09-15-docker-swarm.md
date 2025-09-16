---
layout: post
title: "Docker Swarm"
date: 2025-09-15 15:30:00 +0800
categories: [HomeLab]
tags: [Proxmox, docker]
---
## Intro
This simplified guide for TuringPi cluster is less windy than the original: [rip4cluster](https://rpi4cluster.com/docker-swarm-intro).

# Docker Swarm
Docker Swarm is a native clustering solution for Docker. It lets you run multiple Docker daemons on multiple machines and manage them as a single virtual machine. It allows you to run Docker containers in Highly Available mode, scale them up and down, and run multiple services on the same cluster.

## OS
DietPi lets you quickly set up your system with minimal effort and manual configuration. Docker provides a super light sandboxed application container.
When you export a Docker service in Swarm, any node in the swarm can pick up the connection and relay it to the correct container. Keepalived defines a single IP that can be used for the entire cluster. IT keeps a service running by detecting when a server is not working and redirecting traffic to another working server. This ensures the service is always available to users.

## Layout:
- node1: 10.0.0.21
- node2: 10.0.0.22
- node3: 10.0.0.23
- virtual IP: 10.0.0.20

## Storage
GlusterFS is a scalable, distributed file system that allows you to store and access files across multiple servers. It provides automatic data replication and distribution, ensuring high availability and data redundancy. GlusterFS has advantages over NFS, such as better scalability, higher data protection, and improved performance. The disks should be dedicated to GlusterFS.

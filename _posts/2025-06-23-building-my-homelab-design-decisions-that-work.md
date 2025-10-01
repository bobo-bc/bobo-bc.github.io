---
title: "Building My Homelab: Design Decisions That Work"
slug: building-my-homelab-design-decisions-that-work
date_published: 2025-06-23T18:27:33.000Z
date_updated: 2025-06-23T18:29:11.000Z
tags: Homelab
categories: [homelab]
---

When I first set out to build my homelab, my goal was simple: create a reliable, efficient environment to run the services I rely on, while having full control over my network and infrastructure. After months of experimentation and refinement, here’s a breakdown of the key components and why I chose them.

## **Unifi Gateway Ultra**

I wanted complete visibility and control over my network without compromising on ease of use. Unifi products are well-suited for homelab environments, offering powerful features with a user-friendly interface. My network is powered by the Unifi Gateway Ultra and a Swiss Army-style wireless access point. That’s all I’ve needed to cover my entire home reliably and with excellent performance.

## **Proxmox**

A good lab begins with a solid foundation. Proxmox has proven to be a robust open-source virtualization platform that supports virtual machines, LXC containers, backups, and cloning. It’s the backbone of my infrastructure, allowing me to deploy services flexibly while keeping system resources in check.

## **DietPi OS**

For lightweight VMs and standalone systems, DietPi is my go-to operating system. Its standout feature is the built-in Drive Manager, which automates and simplifies the mounting of local and network drives. It creates proper and reliable fstab entries, which is especially useful when running Docker or Docker Swarm on small-footprint systems.

## **Ubuntu Server**

While a bit more involved to configure than DietPi, Ubuntu Server is well-optimized for Kubernetes (K3s). Once set up, it provides a reliable base for cluster nodes and integrates smoothly with the rest of my stack.

## **TrueNAS Scale (VM)**

Initially, I thought shared storage wouldn’t require much horsepower—until I started running into inode and permission errors using Raspberry Pi-based NFS solutions. I switched to TrueNAS Scale running in a VM with mirrored NVMe drives in a ZFS pool. This setup has been rock-solid, retaining file permissions properly and offering excellent performance for NFS shares used by both Docker and Kubernetes workloads.

## **Plex (LXC)**

For media streaming, I chose to run Plex in an LXC container. LXC allows me to allocate system resources by minimum guarantees rather than maximum limits, which means Plex can burst resources when needed—perfect for transcoding-heavy scenarios without locking up system performance.

## **Pi-hole (LXC and K3s Pod)**

Reliable DNS is essential, especially when using MetalLB for service IP management. I run two Pi-hole instances: the primary is a Kubernetes-managed pod, and the backup runs as an LXC container. A scheduled rsync job ensures their configurations stay synchronized, allowing me to manage only the primary instance while maintaining redundancy.

## **Home Assistant (VM)**

With smart devices all over the house, I didn’t want my home automations to be affected by my homelab experiments. Home Assistant runs as a VM to isolate it from the rest of the stack. After years of using Homebridge, I transitioned to Home Assistant for better compatibility and a more user-friendly interface, especially with newer devices.

## **Jumpbox (VM)**

To manage the cluster, I use a dedicated DietPi-based jumpbox VM. It serves as my main console for installing tools, applying manifests, and managing files. Whether I’m on my laptop or my desktop workstation, I can access and administer my cluster without skipping a beat.

## **Proxmox Backup Server (VM)**

Disasters happen. Having a backup strategy is non-negotiable. My Proxmox Backup Server runs as a VM, capturing snapshots of VMs and containers so I can recover from failures quickly and reliably.

## **GitHub**

All of my infrastructure code—from Docker Swarm YAMLs to Kubernetes manifests—is stored in GitHub. It’s my off-site configuration backup, my version control system, and my deployment hub. If I ever need to rebuild, I can redeploy everything cleanly and confidently.

---

## **Final Thoughts**

Designing a homelab is all about balancing control, reliability, and flexibility. Each tool and technology in my setup was chosen with that balance in mind. This environment not only supports my day-to-day needs but also gives me the freedom to experiment and learn. Whether you’re just getting started or refining your own lab, I hope this breakdown offers a few helpful insights.

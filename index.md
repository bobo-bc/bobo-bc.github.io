---
layout: page
title: Home
permalink: /
toc: true
---

# Homelab

> From a single Debian laptop in 2000 to a production-style Kubernetes platform.

This site documents the evolution of my homelab — not just what I built, but how and why decisions were made along the way.

---

## Overview

I’ve always been drawn to understanding how things work.

The challenge is that most environments don’t allow for experimentation. Over time, I’ve come to believe that the only effective way to learn is to build systems, test them, break them, and rebuild until the solution becomes simple and reliable.

This site is a record of that process.

---

## Architecture and Approach

Containerization provides the foundation — separating workloads and making better use of limited hardware.

From there, clustering introduces resilience and flexibility.

The goal is not just to run services, but to build a system that behaves more like a small-scale production environment. Technologies like Proxmox and Kubernetes make that possible, even within the constraints of a homelab.

---

## Prerequisites

You don’t need to be an expert, but you should be comfortable with:

- The Linux command line  
- Container-based workloads (Docker or similar)  
- Using a GitHub repository to manage configuration  
- Running a small machine with more memory than you initially think you need  

---

## Choose your path

Each series represents a stage in the evolution of this homelab. You can follow them sequentially or focus on a specific area of interest.

---

### Infrastructure

This is the foundation everything else builds on.

Topics include:

- Proxmox virtualization  
- Storage (TrueNAS, NFS, Longhorn)  
- Networking and DNS  

> Start the Infrastructure Series


---

### Docker Swarm

A lightweight introduction to container orchestration.

This stage focuses on understanding how distributed workloads behave, including scheduling, networking, and service management.

>Follow the Swarm Series



---

### k3s Kubernetes

A practical entry point into Kubernetes.

k3s runs well on modest hardware and provides a solid platform for learning how a cluster operates in a real environment.

>Explore the k3s Series



---

### Talos Kubernetes

A more opinionated and production-aligned Kubernetes platform.

With Talos, the operating system becomes part of the platform rather than something managed directly. This is where the homelab begins to feel like a cohesive system.

>Start the Talos Series



---

### Process and ITSM

Building systems is only part of the equation.

This series focuses on how systems are managed, documented, and maintained over time.

>Read the ITSM Series




---

## What you’ll find here

This is not just a collection of tutorials.

It is a record of:

- design decisions  
- tradeoffs and mistakes  
- real-world constraints  
- what I would do differently  

If you are building your own homelab, the goal is to provide clarity and save time through practical experience.

---




---
title: Docker Swarm
icon: network-wired
menu_order: 2
---

# Docker Swarm Guide

Welcome to the Docker Swarm section of my Homelab blog. Here you’ll find simplified instructions and tips for running a highly available Docker Swarm cluster on your TuringPi or other small-scale homelab setup.

---

## Table of Contents

- [Introduction](#introduction)
- [Setting Up the OS](#setting-up-the-os)
- [Deploying Docker Swarm](#deploying-docker-swarm)
- [Networking & Services](#networking--services)
- [References](#references)

---

## Introduction

Docker Swarm is a native clustering solution for Docker. It lets you run multiple Docker daemons on multiple machines and manage them as a single virtual system. You can scale services up and down, and achieve high availability.

---

## Setting Up the OS

I recommend **DietPi** for minimal setup:

- Lightweight, fast, and preconfigured for small clusters
- Works well with Docker
- Minimal manual configuration needed

---

## Deploying Docker Swarm

1. Initialize the swarm on your manager node:

```bash
docker swarm init --advertise-addr <MANAGER-IP>

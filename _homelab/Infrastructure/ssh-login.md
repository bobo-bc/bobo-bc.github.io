---
title: ssh auto login
layout: homelab
series: infrastructure
order: 5
toc: true
---

When building a homelab you spend a lot of time in terminal.  A super simple way of connecting to your remote system is ssh but can be repetitive to authenticate all the time.

prerequisite: - configure your linux os (dietpi) and manually set an IP.  (you should always manually set an ip for a remote server).

I use a Mac laptop so these instructions are MAC centric.

Create an ssh key for your desktop. you only have to do this once.

    ssh-keygen

you will be asked for pass phrase, leave blank for a homelab.

Next, in your terminal copy the newly created key to your linux server

    ssh-copy-id <username@server_ip

It should look something like this

ssh-copy-id dietpi@10.0.0.100

you will be asked to enter the password for the linux username.  once done then type

    ssh dietpi@10.0.0.100

you should now be logged in to the server.

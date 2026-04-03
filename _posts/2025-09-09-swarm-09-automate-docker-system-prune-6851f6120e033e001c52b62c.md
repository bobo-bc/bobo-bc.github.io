---
layout: post
title: Automate docker system prune
date: 2025-09-09
categories: [swarm]
---

Running my docker swarm and experimenting with new things creates a lot of technical debt in the form of unused images, volumes and networks.

log into your primary node and run the following command

    Docker system prune - a -f

This will go through all the container, images (dangling and otherwise) , Volumes and networks. The command will not require interaction.

Since this should be done frequently it makes sense to automate.

go to

    cd /etc/cron.weekly

then type this line

    crontab -e

this will open a crontab file. Scroll to the bottom and add

    0 0 * * 0 docker system prune -a -f

save the file by typing

    control o then enter, then control x

This will run the command once a week at Sunday, midnight.

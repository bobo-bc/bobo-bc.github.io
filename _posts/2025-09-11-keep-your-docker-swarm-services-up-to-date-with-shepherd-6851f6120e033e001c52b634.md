---
title: Shepherd- Keep Your Docker Swarm Services Up-to-Date
date: 2025-09-11 11:40:00 +0800
tags: apps
categories: [swarm]
---

---
# Keeping apps up to date
If you’ve been running a Docker Swarm cluster, you’ve probably realized there’s no built-in mechanism for keeping your services updated when their images change. That’s where [**Shepherd**](https://containrrr.dev/shepherd) comes in—a lightweight tool from the creators of Watchtower, but made especially for Swarm. It keeps an eye on your running services and redeploys them when the image gets updated in your registry.

Let’s walk through how to set up Shepherd in your Swarm cluster and use it to keep your services fresh and shiny.

---

## **What You’ll Need**

- A Docker Swarm cluster (can be a single-node or multi-node setup)
- Docker installed
- Internet access to pull images

---

## **Step 1: Deploy Shepherd in Your Swarm**

Shepherd runs as a global service so that it can observe all nodes in your cluster. Here’s a simple docker-compose.yml to deploy it:

    version: '3.8'
    
    services:
      shepherd:
        image: ghcr.io/containrrr/shepherd:latest
        deploy:
          mode: global
          placement:
            constraints: [node.role == manager]
        volumes:
          - /var/run/docker.sock:/var/run/docker.sock
        environment:
          - SHEPHERD_LOG_LEVEL=info
          - SHEPHERD_POLL_INTERVAL=60s

Save that as shepherd-stack.yml and deploy with:

    docker stack deploy -c shepherd-stack.yml shepherd

This sets up Shepherd to run on all manager nodes and poll your services every 60 seconds.

---

## **Step 2: Label Services to be Watched**

By default, Shepherd doesn’t touch any services. You have to explicitly opt in by adding a label when you deploy them.

Here’s an example of a service you want Shepherd to monitor and auto-redeploy:

    version: '3.8'
    
    services:
      nginx:
        image: nginx:latest
        deploy:
          labels:
            - "shepherd.watch=true"
        ports:
          - 8080:80

Make sure the label is exactly:

    shepherd.watch=true

That’s the trigger for Shepherd to monitor the service’s image and redeploy when it detects a newer one.

---

## **Example Stack Deployment**

Let’s say you have a stack called web-stack with NGINX that you want Shepherd to manage:

    version: '3.8'
    
    services:
      nginx:
        image: nginx:latest
        deploy:
          labels:
            - "shepherd.watch=true"
          replicas: 2
        ports:
          - 80:80

Deploy it:

    docker stack deploy -c web-stack.yml web

Now, whenever there’s a new nginx:latest image, Shepherd will notice and restart the service using the new image.

---

## **Monitoring Shepherd**

You can check Shepherd’s logs using:

    docker service logs shepherd_shepherd

You’ll see entries like:

    Found update for nginx:latest
    Redeploying service web_nginx

Or better yet, have a look at ghcr.io/sergi0g/cup a simple app that I have deployed to swarm and all it does is report in a browser if apps are up to date.  This was very useful during Shephard troubleshooting.

---

## **Pro Tips**

- Use immutable image tags like myapp:2025.05.13 if you want more control over what gets updated.
- Shepherd does not rebuild images—just watches for newer ones.
- Works best when you have CI pipelines pushing fresh images regularly.

---

## **Clean Up**

To remove Shepherd:

    docker stack rm shepherd

---

## **Final Thoughts**

Shepherd is a “set it and forget it” tool that takes the pain out of keeping your Docker Swarm services updated. It’s ideal for setups where you’re using :latest or rolling tags and want your containers to stay in sync with the image registry.

If you’re already using Swarm in your home lab or even a production microservices setup, this is one of those tools that’s worth adding to your stack.

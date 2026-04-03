---
title: "Automating Docker Swarm Deployments:"
date: 2025-09-12 11:40:00 +0800
tags: github
categories: [swarm]
---

# Portainer and Github for Swarm CI/CD

## **Introduction**

Managing **Docker Swarm stacks** efficiently requires proper version control and automation. In this guide, you'll learn how to: 

✅ **Set up a GitHub repository** for your Docker Swarm configuration. 

✅ **Deploy a stack in Portainer EE** directly from the GitHub repository. 

✅ **Automate deployments** for seamless updates and improvements.

By following these steps, you’ll enable **automated stack deployment and version tracking** in a production-ready Swarm environment.

--

# **Set Up a GitHub Repository for Docker Compose Files**

Before deploying with **Portainer EE**, you need a **GitHub repository** to host your stack configurations.

### **Create a GitHub Repository**

1️⃣ Log into **GitHub** and navigate to GitHub Repositories. 

2️⃣ Click **New repository** and name it (e.g., `docker-swarm-stacks`). 

3️⃣ Set visibility (**Private** or **Public**). 

4️⃣ Click **Create repository**.

### **Add Your Docker Compose File**

Create a **Docker Swarm stack definition** inside your repository.

#### **Example: **`docker-compose.yml`

yaml

    version: "3.9"
    
    services:
      app:
        image: nginx
        ports:
          - "80:80"
        deploy:
          replicas: 3
          restart_policy:
            condition: on-failure
    

### **Commit and Push to GitHub**

Upload your file to GitHub from your local machine:

bash

    git init
    git add docker-compose.yml
    git commit -m "Initial stack configuration"
    git branch -M main
    git remote add origin 
    git push -u origin main
--    

## **Configure Portainer-ee (Enterprise) to Deploy from GitHub**

### **Log into Portainer-ee**

✅ Open your **Portainer-ee** dashboard.

✅ Navigate to your **Swarm environment**.

### **Set Up GitHub as an External Repository**

1. Go to **Stacks** → **Add a Stack**. 
2. Choose **Git Repository** as the deployment source.
3. Enter your **GitHub repository URL**:
4. Specify the **branch** (`main`) and file (`docker-compose.yml`).
5. Enter **authentication credentials** if using a private repository.

### **Deploy the Stack**

Click **Deploy the stack** to initiate the deployment process.
--

#Automate Stack Updates**

To ensure **automatic redeployment** of updated stacks:

✅ **Enable Webhooks** in GitHub to notify Portainer EE. 

✅ Set a **Cron Job** to pull latest changes in Portainer EE. 

✅ Use **Watchtower** to monitor image updates.

Example automation using Portainer API:

bash

    curl -X POST http://portainer.example.com/api/stacks/redeploy -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
    
--
## **Conclusion**

By integrating **GitHub with Portainer EE**, you’ve built a **scalable, automated deployment pipeline**. Your stacks remain version-controlled, easily updated, and secure.

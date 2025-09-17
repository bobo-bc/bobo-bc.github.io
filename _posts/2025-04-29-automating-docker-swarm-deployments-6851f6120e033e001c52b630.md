---
title: "Automating Docker Swarm Deployments:"
slug: automating-docker-swarm-deployments-6851f6120e033e001c52b630
date_published: 2025-04-30T02:25:07.000Z
date_updated: 2025-04-30T02:25:07.000Z
tags: Docker Swarm, #Migrated-1750201873783, #wp, #wp-post, #Import 2025-06-17 16:11
---

**Using GitHub as a Repository and Deploying with Portainer EE**

## **Introduction**

Managing **Docker Swarm stacks** efficiently requires proper version control and automation. In this guide, you'll learn how to: ✅ **Set up a GitHub repository** for your Docker Swarm configuration. ✅ **Deploy a stack in Portainer EE** directly from the GitHub repository. ✅ **Automate deployments** for seamless updates and improvements.

By following these steps, you’ll enable **automated stack deployment and version tracking** in a production-ready Swarm environment.

## **Step 1: Set Up a GitHub Repository for Docker Compose Files**

Before deploying with **Portainer EE**, you need a **GitHub repository** to host your stack configurations.

### **1. Create a GitHub Repository**

1️⃣ Log into **GitHub** and navigate to GitHub Repositories. 2️⃣ Click **New repository** and name it (e.g., `docker-swarm-stacks`). 3️⃣ Set visibility (**Private** or **Public**). 4️⃣ Click **Create repository**.

### **2. Add Your Docker Compose File**

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
    

### **3. Commit and Push to GitHub**

Upload your file to GitHub from your local machine:

bash

    git init
    git add docker-compose.yml
    git commit -m "Initial stack configuration"
    git branch -M main
    git remote add origin https://github.com/yourusername/docker-swarm-stacks.git
    git push -u origin main
    

## **Step 2: Configure Portainer EE to Deploy from GitHub**

### **1. Log into Portainer EE**

✅ Open your **Portainer EE** dashboard. ✅ Navigate to your **Swarm environment**.

### **2. Set Up GitHub as an External Repository**

1️⃣ Go to **Stacks** → **Add a Stack**. 2️⃣ Choose **Git Repository** as the deployment source. 3️⃣ Enter your **GitHub repository URL**:

    https://github.com/yourusername/docker-swarm-stacks.git
    

4️⃣ Specify the **branch** (`main`) and file (`docker-compose.yml`). 5️⃣ Enter **authentication credentials** if using a private repository.

### **3. Deploy the Stack**

Click **Deploy the stack** to initiate the deployment process.

## **Step 3: Automate Stack Updates**

To ensure **automatic redeployment** of updated stacks:

✅ **Enable Webhooks** in GitHub to notify Portainer EE. ✅ Set a **Cron Job** to pull latest changes in Portainer EE. ✅ Use **Watchtower** to monitor image updates.

Example automation using Portainer API:

bash

    curl -X POST http://portainer.example.com/api/stacks/redeploy -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
    

## **Conclusion**

By integrating **GitHub with Portainer EE**, you’ve built a **scalable, automated deployment pipeline**. Your stacks remain version-controlled, easily updated, and secure.

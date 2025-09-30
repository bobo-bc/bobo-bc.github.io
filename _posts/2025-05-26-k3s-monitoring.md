---
title: Monitoring and Performance in k3s
slug: k3s-monitoring
date_published: 2025-05-27T00:16:13.000Z
date_updated: 2025-06-18T18:18:27.000Z
tags: Kubernetes
---

**with Prometheus, Grafana, and Homarr**

As your home Kubernetes cluster grows, so does the need to keep tabs on its health, performance, and resource usage. In this post, we’ll walk through setting up a robust monitoring stack using Prometheus and Grafana, with bonus integration into the Homarr dashboard.

---

## Stack Overview

We’ll be using:

- **Prometheus**: For collecting metrics
- **Grafana**: For visualizing metrics
- **Kube State Metrics** and **Node Exporter**: For detailed cluster data
- **MetalLB**: To expose the Grafana UI
- **Homarr**: To view widgets from Prometheus

## Step-by-Step Instructions

### 1. Add the Prometheus Helm Repo

SSH into your jumpbox and run:

    helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
    helm repo update
    

### 2. Install kube-prometheus-stack

This will install everything we need:

    helm upgrade --install monitoring prometheus-community/kube-prometheus-stack \
      --namespace monitoring --create-namespace \
      --set prometheus.prometheusSpec.maximumStartupDurationSeconds=300
    

This includes:

- Prometheus
- Grafana
- AlertManager
- Kube State Metrics
- Node Exporter

### 3. Expose Grafana with MetalLB

Create a `grafana-service.yaml`:

    apiVersion: v1
    kind: Service
    metadata:
      name: grafana-lb
      namespace: monitoring
    spec:
      type: LoadBalancer
      selector:
        app.kubernetes.io/name: grafana
      ports:
        - port: 80
          targetPort: 3000
          protocol: TCP
          name: http
      loadBalancerIP: 10.0.0.40
    

Apply it:

    kubectl apply -f grafana-service.yaml
    

### 4. Get the Grafana Admin Password

    kubectl get secret --namespace monitoring monitoring-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
    

Default user: `admin`

Access Grafana at `http://10.0.0.40`

### 5. Add Prometheus Widgets to Homarr

If you already have Homarr installed, add widgets by editing your config and using Prometheus as a data source:

    - type: prometheus
      title: CPU Usage
      endpoint: http://monitoring-kube-prometheus-prometheus.monitoring.svc.cluster.local:9090
      query: sum(rate(container_cpu_usage_seconds_total[1m]))
      format: short
    

Note: You can expose Prometheus via MetalLB or a Traefik ingress if needed.

---

## Useful Dashboards to Import in Grafana

- Kubernetes Cluster Monitoring (ID: 315)
- Node Exporter Full (ID: 1860)
- Longhorn Dashboard (search on Grafana.com)

---

## ? Recap

With Prometheus and Grafana, you now have:

- Live monitoring of CPU, memory, pods, and nodes
- Dashboard access via MetalLB IP
- Visibility from your Homarr homepage

This closes the loop on not just deploying workloads in k3s, but operating them confidently.

Up next: we’ll explore setting up Alerts and Notification hooks using AlertManager.

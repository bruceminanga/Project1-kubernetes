# Kubernetes Fundamentals: NGINX Web Server Deployment

<div align="center">

![Kubernetes](https://img.shields.io/badge/kubernetes-%23326ce5.svg?style=for-the-badge&logo=kubernetes&logoColor=white)
![NGINX](https://img.shields.io/badge/nginx-%23009639.svg?style=for-the-badge&logo=nginx&logoColor=white)
![Docker](https://img.shields.io/badge/docker-%230db7ed.svg?style=for-the-badge&logo=docker&logoColor=white)

**Master Kubernetes fundamentals by deploying a highly available NGINX web server**

[Quick Start](#quick-start) •
[Documentation](#step-by-step-guide) •
[Operations](#operations--management) •
[Troubleshooting](#troubleshooting)

</div>

---

## 🎯 Project Overview

This hands-on project demonstrates Kubernetes fundamentals by deploying a production-ready NGINX web server with high availability. You'll learn essential Kubernetes concepts while building a scalable, resilient application that automatically recovers from failures.

### What You'll Build

- 🚀 **Highly Available Web Server**: 3 NGINX replicas for redundancy
- 🔄 **Self-Healing Application**: Automatic pod recovery and health monitoring
- 📊 **Load Balanced Traffic**: Service distributes requests across healthy pods
- ⚡ **Easy Scaling**: Scale up/down with a single command

## 🧠 Core Concepts Covered

| Component              | Purpose                      | Key Learning                           |
| ---------------------- | ---------------------------- | -------------------------------------- |
| **Pods**               | Smallest deployable units    | Foundation of Kubernetes workloads     |
| **Deployments**        | Manage application lifecycle | Rolling updates, scaling, self-healing |
| **Services**           | Stable network endpoints     | Load balancing and service discovery   |
| **Labels & Selectors** | Resource organization        | How Kubernetes connects components     |

## 📋 Prerequisites

### Required Tools

- **Kubernetes Cluster**:
  - [Minikube](https://minikube.sigs.k8s.io/docs/start/) (recommended)
  - [Kind](https://kind.sigs.k8s.io/)
  - Docker Desktop with Kubernetes
- **kubectl**: [Install kubectl](https://kubernetes.io/docs/tasks/tools/)
- **Web Browser**: Any modern browser

### Knowledge

- Basic command line experience
- Container concepts (helpful but not required)

## ⚡ Quick Start

```bash
# 1. Start your cluster
minikube start

# 2. Deploy the application
kubectl apply -f my-webapp.yaml

# 3. Access your web server
minikube service my-webapp-service
```

🎉 **That's it!** Your NGINX server is now running with high availability on Kubernetes.

## 📖 Step-by-Step Guide

### Step 1: Initialize Your Cluster

```bash
# Start minikube
minikube start

# Verify cluster is ready
kubectl get nodes
```

**Expected Output:**

```
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   1m    v1.28.3
```

### Step 2: Create the Application Manifest

Create `my-webapp.yaml`:

```yaml
---
# Deployment: Manages our application pods
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-webapp
  labels:
    app: my-webapp
    version: "1.0"
spec:
  replicas: 3 # High availability with 3 replicas
  selector:
    matchLabels:
      app: my-webapp
  template:
    metadata:
      labels:
        app: my-webapp
        version: "1.0"
    spec:
      containers:
        - name: nginx-container
          image: nginx:1.21-alpine
          ports:
            - containerPort: 80
          resources:
            requests:
              memory: "64Mi"
              cpu: "50m"
            limits:
              memory: "128Mi"
              cpu: "100m"
          # Health checks for reliability
          livenessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 30
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 5
            periodSeconds: 5

---
# Service: Provides stable network access
apiVersion: v1
kind: Service
metadata:
  name: my-webapp-service
  labels:
    app: my-webapp
spec:
  type: NodePort
  selector:
    app: my-webapp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

### Step 3: Deploy Your Application

```bash
# Apply the configuration
kubectl apply -f my-webapp.yaml

# Verify deployment
kubectl get deployments
kubectl get pods
kubectl get services
```

### Step 4: Access Your Application

```bash
# Open in browser (minikube)
minikube service my-webapp-service

# Alternative: Get the URL
minikube service my-webapp-service --url
```

## ⚙️ Operations & Management

### Scaling Your Application

```bash
# Scale up for increased traffic
kubectl scale deployment my-webapp --replicas=5

# Scale down to save resources
kubectl scale deployment my-webapp --replicas=2

# Verify scaling
kubectl get pods
```

### Monitoring & Debugging

```bash
# View application logs
kubectl logs deployment/my-webapp

# Follow logs in real-time
kubectl logs -f deployment/my-webapp

# Check resource usage
kubectl top pods

# Inspect pod details
kubectl describe pods -l app=my-webapp
```

### Rolling Updates

```bash
# Update to new NGINX version
kubectl set image deployment/my-webapp nginx-container=nginx:1.22-alpine

# Monitor the rollout
kubectl rollout status deployment/my-webapp

# Rollback if needed
kubectl rollout undo deployment/my-webapp
```

## 🏗️ Architecture

```
                    Internet
                       │
                       ▼
    ┌─────────────────────────────────────┐
    │          Minikube Node              │
    │                                     │
    │    ┌─────────────────────────┐      │
    │    │   my-webapp-service     │      │
    │    │    (Load Balancer)      │      │
    │    │       Port 80           │      │
    │    └──────────┬──────────────┘      │
    │               │                     │
    │    ┌──────────┼──────────┐          │
    │    │          │          │          │
    │    ▼          ▼          ▼          │
    │ ┌──────┐  ┌──────┐  ┌──────┐       │
    │ │Pod 1 │  │Pod 2 │  │Pod 3 │       │
    │ │NGINX │  │NGINX │  │NGINX │       │
    │ │ :80  │  │ :80  │  │ :80  │       │
    │ └──────┘  └──────┘  └──────┘       │
    └─────────────────────────────────────┘
```

## 🔧 Troubleshooting

### Common Issues & Solutions

**Pods not starting:**

```bash
kubectl get pods
kubectl describe pod <pod-name>
kubectl get events --sort-by='.lastTimestamp'
```

**Service not accessible:**

```bash
kubectl get services
kubectl describe service my-webapp-service
kubectl get endpoints
```

**Image pull errors:**

```bash
kubectl describe pod <pod-name>
# Check the Events section for image pull failures
```

### Useful Debug Commands

```bash
# Get cluster information
kubectl cluster-info

# View all resources
kubectl get all

# Check node status
kubectl describe nodes

# Execute commands in a pod
kubectl exec -it deployment/my-webapp -- /bin/sh
```

## 🧹 Cleanup

```bash
# Remove the application
kubectl delete -f my-webapp.yaml

# Stop the cluster (optional)
minikube stop

# Delete cluster entirely (optional)
minikube delete
```

## 🚀 Next Steps

Ready to advance your Kubernetes skills? Try these concepts:

- **Persistent Storage**: Add volumes for data persistence
- **ConfigMaps & Secrets**: Externalize configuration
- **Ingress Controllers**: Set up advanced routing
- **Helm Charts**: Package applications for easy deployment
- **Monitoring**: Add Prometheus and Grafana

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

---

<div align="center">

**⭐ Star this repo if it helped you learn Kubernetes! ⭐**

_Happy Kubernetes learning! 🎉_

</div>

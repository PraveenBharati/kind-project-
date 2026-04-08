# K8s Demo App — Kind Deployment Guide

## Project Structure

```
k8s-demo-app/
├── frontend/
│   ├── index.html       # Simple HTML/JS UI
│   ├── nginx.conf       # Nginx reverse proxy config
│   └── Dockerfile
├── backend/
│   ├── server.js        # Node.js REST API
│   ├── package.json
│   └── Dockerfile
└── k8s/
    ├── kind-cluster.yaml   # Kind cluster definition
    ├── postgres.yaml        # DB: Secret, PVC, Deployment, Service
    ├── backend.yaml         # API: Deployment, Service
    └── frontend.yaml        # UI: Deployment, NodePort Service
```

## Architecture

```
Browser (localhost:8080)
    │
    ▼
[frontend-service NodePort:30080]
    │ Nginx proxies /api/* to backend
    ▼
[backend-service ClusterIP:3000]
    │ pg client
    ▼
[postgres-service ClusterIP:5432]
```

---

## Prerequisites

Make sure these are installed on your machine:
- Docker
- Kind: https://kind.sigs.k8s.io/docs/user/quick-start/#installation
- kubectl: https://kubernetes.io/docs/tasks/tools/

---

## Step-by-Step Deployment

### 1. Create the Kind Cluster

```bash
kind create cluster --config k8s/kind-cluster.yaml
```

Verify:
```bash
kubectl cluster-info --context kind-demo-cluster
```

---

### 2. Build Docker Images

```bash
# Build frontend image
docker build -t k8s-demo-frontend:latest ./frontend

# Build backend image
docker build -t k8s-demo-backend:latest ./backend
```

---

### 3. Load Images into Kind

Kind cannot pull local Docker images automatically — you must load them:

```bash
kind load docker-image k8s-demo-frontend:latest --name demo-cluster
kind load docker-image k8s-demo-backend:latest --name demo-cluster
```

---

### 4. Deploy to Kubernetes

Apply manifests in order:

```bash
# 1. Deploy PostgreSQL (DB first)
kubectl apply -f k8s/postgres.yaml

# 2. Wait for Postgres to be ready
kubectl wait --for=condition=ready pod -l app=postgres --timeout=60s

# 3. Deploy Backend
kubectl apply -f k8s/backend.yaml

# 4. Deploy Frontend
kubectl apply -f k8s/frontend.yaml
```

---

### 5. Verify Everything is Running

```bash
kubectl get pods
kubectl get services
```

Expected output:
```
NAME                        READY   STATUS    RESTARTS
postgres-xxxx               1/1     Running   0
backend-xxxx                1/1     Running   0
frontend-xxxx               1/1     Running   0
```

---

### 6. Access the App

Open your browser at:
```
http://localhost:8080
```

You should see the demo UI where you can:
- Check backend health
- Add users (stored in PostgreSQL)
- List users from the DB

---

## Useful Commands

```bash
# View logs
kubectl logs -l app=backend
kubectl logs -l app=frontend
kubectl logs -l app=postgres

# Describe a pod (for debugging)
kubectl describe pod -l app=backend

# Delete everything
kubectl delete -f k8s/
kind delete cluster --name demo-cluster
```

---

## Cleanup

```bash
kind delete cluster --name demo-cluster
```

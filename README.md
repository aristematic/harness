# Harness CI/CD Demo

End-to-end CI/CD pipeline using Harness — builds a Node.js app, pushes to DockerHub, and deploys to a local Kubernetes cluster (kind).

## Pipeline Overview
```
CI-Build → approval-gate → CD-Deploy
```

| Stage | What it does |
|---|---|
| CI-Build | Clones repo, runs tests, builds Docker image, pushes to DockerHub |
| approval-gate | Manual approval gate before deployment |
| CD-Deploy | Pulls image from DockerHub, deploys to Kubernetes via Rolling update |

## Tech Stack

- **Harness** — CI/CD platform
- **Docker** — containerization
- **Kubernetes (kind)** — local cluster for deployment
- **Node.js** — sample application
- **GitHub** — source code + manifest store

## Project Structure
```
harness-demo/
├── app.js                  # Node.js app (runs on port 3000)
├── Dockerfile              # Container image definition
├── k8s/
│   └── deployment.yaml     # Kubernetes Deployment + Service
├── harness/
│   └── pipeline.yaml       # Harness pipeline as code
└── README.md
```

## Prerequisites

- [kind](https://kind.sigs.k8s.io/) — local Kubernetes cluster
- [kubectl](https://kubernetes.io/docs/tasks/tools/) — Kubernetes CLI
- [Docker](https://www.docker.com/) — container runtime
- [Harness account](https://app.harness.io) — free tier works
- DockerHub account — for image registry

## Setup

### 1. Create kind cluster
```bash
kind create cluster --name harness
```

### 2. Install Harness Delegate
```bash

# In Harness UI:
# Project Settings → Delegates → + New Delegate → Kubernetes → Helm
# Copy the helm command and run it
helm install helm-delegate \
  harness-delegate/harness-delegate \
  --namespace harness-delegate \
  --create-namespace \
  [other flags from Harness UI]

# Verify delegate is running
kubectl get pods -n harness-delegate
```
### 3. Configure Connectors in Harness UI
```
Project Settings → Connectors:
  - github-connector    → your GitHub account
  - dockerhub-connector → your DockerHub account
  - k8s-connector       → your kind cluster via delegate
```

### 4. Create ImagePullSecret for private DockerHub repo
```bash
kubectl create secret docker-registry dockerhub-secret \
  --docker-username=YOUR_DOCKERHUB_USERNAME \
  --docker-password=YOUR_DOCKERHUB_TOKEN \
  --docker-email=YOUR_EMAIL \
  -n default
```

### 5. Run the Pipeline
```
Harness UI → Pipelines → harness-pipeline → Run
- Branch: main
- Artifact Tag: latest
```

## Application

Simple Node.js HTTP server:
```JavaScript
// responds on port 3000
Hello from Harness CI/CD! v1.0
```

## Verify Deployment
```bash
# Check pod is running
kubectl get pods -n default

# Port forward
kubectl port-forward svc/harness-svc 9090:80

# Hit the app
curl http://localhost:9090
# Hello from Harness CI/CD! v1.0
```

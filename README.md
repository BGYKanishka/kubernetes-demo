# Kubernetes Demo API

A simple Node.js (Express) REST API demonstrating how to containerize an app with Docker and deploy it to Kubernetes using Minikube.

---

## Table of Contents

- [What This Project Does](#what-this-project-does)
- [Prerequisites](#prerequisites)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
  - [1. Clone the Repository](#1-clone-the-repository)
  - [2. Install Dependencies](#2-install-dependencies)
  - [3. Run Locally (No Docker)](#3-run-locally-no-docker)
- [Running with Docker](#running-with-docker)
  - [Option A — Docker Compose (Recommended for Dev)](#option-a--docker-compose-recommended-for-dev)
  - [Option B — Plain Docker](#option-b--plain-docker)
- [Deploying to Kubernetes (Minikube)](#deploying-to-kubernetes-minikube)
  - [Step 1 — Install Minikube & kubectl](#step-1--install-minikube--kubectl)
  - [Step 2 — Start the Cluster](#step-2--start-the-cluster)
  - [Step 3 — Create the Kubernetes Secret](#step-3--create-the-kubernetes-secret)
  - [Step 4 — Deploy](#step-4--deploy)
  - [Step 5 — Access the App](#step-5--access-the-app)
- [API Endpoints](#api-endpoints)
- [Useful Commands](#useful-commands)
- [Troubleshooting](#troubleshooting)

---

## What This Project Does

Returns a simple JSON response with the pod name, service info, and current time. It includes health-check endpoints (`/healthz`, `/readyz`) used by Kubernetes to monitor the container.

**Example response from `GET /`:**

```json
{
  "message": "Hello form container",
  "service": "hello from node",
  "pod": "kubernetes-demo-api-6f7b8c9d-xk4jn",
  "time": "2026-09-05T19:07:25.000Z"
}
```

---

## Prerequisites

Make sure you have the following installed **before** starting:

| Tool | Version | Install Link |
|------|---------|--------------|
| **Node.js** | 18+ | [nodejs.org](https://nodejs.org/) |
| **npm** | 9+ | Comes with Node.js |
| **Docker Desktop** | Latest | [docker.com/get-started](https://www.docker.com/get-started/) |
| **Minikube** | Latest | [minikube.sigs.k8s.io](https://minikube.sigs.k8s.io/docs/start/) |
| **kubectl** | Latest | [kubernetes.io/docs/tasks/tools](https://kubernetes.io/docs/tasks/tools/) |
| **Docker Hub Account** | — | [hub.docker.com](https://hub.docker.com/) (only needed if you want to push your own image) |

> **Tip:** On macOS you can install Minikube and kubectl with Homebrew:
> ```bash
> brew install minikube kubectl
> ```

---

## Project Structure

```
kubernetes-demo/
├── index.js                 # Express API entry point
├── package.json             # Node.js dependencies & scripts
├── Dockerfile               # Container image definition
├── docker-compose.yaml      # Docker Compose config (dev mode)
├── deploy.sh                # One-command build + push + deploy script
├── .dockerignore             # Files excluded from Docker build
├── .env.local               # Local environment variables (DB connection)
├── k8s/
│   ├── deployment.yaml      # Kubernetes Deployment (2 replicas)
│   └── service.yaml         # Kubernetes Service (NodePort)
└── README.md                # ← You are here
```

---

## Getting Started

### 1. Clone the Repository

```bash
git clone https://github.com/yehankanishka/kubernetes-demo.git
cd kubernetes-demo
```

### 2. Install Dependencies

```bash
npm install
```

### 3. Run Locally (No Docker)

```bash
npm run dev
```

The server starts at **http://localhost:3000** with live-reload enabled (`--watch` flag).

Open your browser or use `curl`:

```bash
curl http://localhost:3000
```

---

## Running with Docker

### Option A — Docker Compose (Recommended for Dev)

This mounts your source code into the container so changes are reflected immediately.

```bash
docker compose up --build
```

The app will be available at **http://localhost:3000**.

To stop:

```bash
docker compose down
```

### Option B — Plain Docker

Build and run the production image manually:

```bash
# Build the image
docker build -t kubernetes-demo-api .

# Run the container
docker run -p 3000:3000 kubernetes-demo-api
```

App available at **http://localhost:3000**.

---

## Deploying to Kubernetes (Minikube)

### Step 1 — Install Minikube & kubectl

**macOS (Homebrew):**

```bash
brew install minikube kubectl
```

**Other platforms:** Follow the official guides:
- Minikube: https://minikube.sigs.k8s.io/docs/start/
- kubectl: https://kubernetes.io/docs/tasks/tools/

Verify installation:

```bash
minikube version
kubectl version --client
```

### Step 2 — Start the Cluster

```bash
minikube start
```

Wait until you see `Done! kubectl is now configured to use "minikube" cluster`.

> **⚠️ Important:** Minikube must be running before you deploy. If you see
> `connection refused` errors, run `minikube start` first.

### Step 3 — Create the Kubernetes Secret

The app expects a `DATABASE_URL` secret. Create it from your `.env.local` values:

```bash
kubectl create secret generic api-secrets \
  --from-literal=DATABASE_URL="<your-database-url-here>"
```

> Replace `<your-database-url-here>` with your actual database connection string from `.env.local`.

To verify the secret was created:

```bash
kubectl get secrets
```

### Step 4 — Deploy

**Option A — One Command (build + push + deploy):**

> **Note:** This pushes to Docker Hub under the `yehankanishka` account. Update `deploy.sh` with your own Docker Hub username if you are forking this project.

```bash
# Make sure you are logged in to Docker Hub
docker login

# Make sure Minikube is running
minikube status

# Deploy everything
npm run deploy
```

**Option B — Step by Step:**

```bash
# 1. Build the Docker image
docker build -t yehankanishka/kubernetes-demo-api:latest .

# 2. Push to Docker Hub
docker push yehankanishka/kubernetes-demo-api:latest

# 3. Apply the Kubernetes manifests
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
```

### Step 5 — Access the App

After deploying, get the URL for the service:

```bash
minikube service devops-kubernetes-api-service --url
```

This prints a URL like `http://192.168.49.2:31234` — open it in your browser or `curl` it.

Alternatively, forward the port to localhost:

```bash
kubectl port-forward service/devops-kubernetes-api-service 3000:3000
```

Now access it at **http://localhost:3000**.

---

## API Endpoints

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/` | Returns JSON with message, service name, pod name, and timestamp |
| `GET` | `/healthz` | Liveness probe — returns `ok` (used by Kubernetes) |
| `GET` | `/readyz` | Readiness probe — returns `ready` (used by Kubernetes) |

---

## Useful Commands

```bash
# Check cluster status
minikube status

# View running pods
kubectl get pods

# View services
kubectl get services

# View pod logs
kubectl logs -l app=kubernetes-demo-api

# Follow logs in real time
kubectl logs -f -l app=kubernetes-demo-api

# Describe a pod (debugging)
kubectl describe pod <pod-name>

# Scale the deployment (e.g., to 3 replicas)
kubectl scale deployment kubernetes-demo-api --replicas=3

# Delete everything and start fresh
kubectl delete -f k8s/deployment.yaml
kubectl delete -f k8s/service.yaml

# Stop Minikube (preserves cluster state)
minikube stop

# Delete Minikube cluster entirely
minikube delete
```

---

## Troubleshooting

### `connection refused` when running `kubectl` commands

**Cause:** Minikube is not running.

```bash
minikube start
```

### Pods stuck in `CrashLoopBackOff`

**Cause:** Usually a missing secret or environment variable.

```bash
# Check the logs
kubectl logs -l app=kubernetes-demo-api

# Make sure the secret exists
kubectl get secrets
```

### `ImagePullBackOff` error

**Cause:** Kubernetes can't pull the Docker image.

- Make sure the image is pushed to Docker Hub: `docker push yehankanishka/kubernetes-demo-api:latest`
- Make sure the image name in `k8s/deployment.yaml` matches your Docker Hub repo.

### Port already in use

**Cause:** Another process is using port 3000.

```bash
# Find the process
lsof -i :3000

# Kill it
kill -9 <PID>
```

---

## License

ISC

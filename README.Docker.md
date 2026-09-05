# Docker Notes

> This file contains Docker-specific notes. For the full setup guide (local dev, Docker, and Kubernetes), see [README.md](./README.md).

## Quick Reference

```bash
# Build and run with Docker Compose (dev mode with live-reload)
docker compose up --build

# Build production image
docker build -t kubernetes-demo-api .

# Run production container
docker run -p 3000:3000 kubernetes-demo-api

# Stop Docker Compose
docker compose down
```

## Cross-Platform Builds

If deploying to a cloud provider with a different CPU architecture (e.g., building on Apple Silicon for an amd64 cloud):

```bash
docker build --platform=linux/amd64 -t kubernetes-demo-api .
```

## Further Reading

- [Docker Getting Started — Sharing](https://docs.docker.com/go/get-started-sharing/)
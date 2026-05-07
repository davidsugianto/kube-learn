# Prerequisites

Before diving into Kubernetes, ensure you have a solid understanding of containers and the necessary tools installed.

## Container Basics

Kubernetes orchestrates containers. Understanding containers is essential.

### Key Concepts

- **Images**: Read-only templates used to create containers
- **Containers**: Running instances of images with isolated filesystem, network, and processes
- **Registries**: Storage for container images (Docker Hub, GHCR, ECR, GCR)

### Container Runtimes

Kubernetes supports multiple container runtimes via CRI (Container Runtime Interface):

| Runtime | Description |
|---------|-------------|
| containerd | Industry standard, most common |
| CRI-O | Lightweight, OCI-focused |
| Docker Desktop | Uses containerd internally |

## Required Tools

### kubectl

The Kubernetes command-line tool.

```bash
# macOS
brew install kubectl

# Linux
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/

# Verify
kubectl version --client
```

### Local Kubernetes Options

| Tool | Best For | Resource Usage |
|------|----------|----------------|
| **minikube** | Full-featured local K8s | Medium-High |
| **kind** (Kubernetes in Docker) | CI/CD, multi-cluster | Medium |
| **k3d** (k3s in Docker) | Lightweight, fast | Low |
| **Docker Desktop** | Easiest setup, all-in-one | High |
| **MicroK8s** | Ubuntu/Canonical ecosystem | Medium |

### Installation Examples

**minikube:**
```bash
# macOS
brew install minikube

# Start cluster
minikube start

# Use Docker driver (recommended)
minikube start --driver=docker
```

**kind:**
```bash
# macOS
brew install kind

# Create cluster
kind create cluster

# With config
kind create cluster --config kind-config.yaml
```

**k3d:**
```bash
# macOS
brew install k3d

# Create cluster
k3d cluster create mycluster
```

## Useful Tools

### Development

```bash
# Helm - Package manager
brew install helm

# k9s - Terminal UI
brew install k9s

# stern - Multi-pod logs
brew install stern

# krew - kubectl plugin manager
brew install krew
```

### Validation

```bash
# kubeval - Validate manifests
brew install kubeval

# kube-score - Linter
brew install kube-score
```

## Verify Your Setup

```bash
# Check kubectl
kubectl version --client

# Check cluster access
kubectl cluster-info

# Check nodes
kubectl get nodes

# Run a test deployment
kubectl create deployment nginx --image=nginx
kubectl get pods
kubectl delete deployment nginx
```

## Next Steps

Once you have a working cluster and can run basic kubectl commands, proceed to [01-fundamentals](../01-fundamentals).

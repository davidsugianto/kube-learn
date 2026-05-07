# Prerequisites - Commands

## Docker/Container Commands

```bash
# List containers
docker ps -a

# Build image
docker build -t myimage:v1 .

# Run container
docker run -d -p 8080:80 nginx

# View logs
docker logs <container-id>

# Execute in container
docker exec -it <container-id> sh

# Clean up
docker system prune -a
```

## minikube Commands

```bash
# Start cluster
minikube start

# Start with specific k8s version
minikube start --kubernetes-version=v1.28.0

# Start with resources
minikube start --memory=4096 --cpus=2

# Stop cluster
minikube stop

# Delete cluster
minikube delete

# SSH into node
minikube ssh

# Open dashboard
minikube dashboard

# Use local docker images
eval $(minikube docker-env)

# List addons
minikube addons list

# Enable addon
minikube addons enable ingress
```

## kind Commands

```bash
# Create cluster
kind create cluster

# Create with name
kind create cluster --name my-cluster

# Delete cluster
kind delete cluster

# List clusters
kind get clusters

# Load image into cluster
kind load docker-image myimage:v1

# Export logs
kind export logs ./logs
```

## k3d Commands

```bash
# Create cluster
k3d cluster create mycluster

# Create with ports
k3d cluster create mycluster -p "8080:80@loadbalancer"

# List clusters
k3d cluster list

# Delete cluster
k3d cluster delete mycluster

# Import image
k3d image import myimage:v1 -c mycluster
```

## kubectl Basic Commands

```bash
# Check connection
kubectl cluster-info

# Get resources
kubectl get nodes
kubectl get pods -A
kubectl get deployments -A

# Describe resource
kubectl describe node <node-name>

# View logs
kubectl logs <pod-name>

# Execute in pod
kubectl exec -it <pod-name> -- sh

# Apply manifest
kubectl apply -f manifest.yaml

# Delete resource
kubectl delete -f manifest.yaml
```

## Context & Config

```bash
# View config
kubectl config view

# List contexts
kubectl config get-contexts

# Switch context
kubectl config use-context <context-name>

# Current context
kubectl config current-context
```

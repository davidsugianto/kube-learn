# Fundamentals - Commands

## Cluster Information

```bash
# Cluster info
kubectl cluster-info

# Cluster dump (debugging)
kubectl cluster-info dump

# View kubeconfig
kubectl config view

# Get contexts
kubectl config get-contexts

# Switch context
kubectl config use-context <context-name>
```

## Node Operations

```bash
# List nodes
kubectl get nodes

# Wide output
kubectl get nodes -o wide

# Describe node
kubectl describe node <node-name>

# Node resource usage
kubectl top node

# Cordon (mark unschedulable)
kubectl cordon <node-name>

# Uncordon (mark schedulable)
kubectl uncordon <node-name>

# Drain (evict pods)
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data
```

## Namespace Operations

```bash
# List namespaces
kubectl get namespaces

# Create namespace
kubectl create namespace <name>

# Delete namespace
kubectl delete namespace <name>

# Set default namespace
kubectl config set-context --current --namespace=<name>

# Get all in namespace
kubectl get all -n <namespace>
```

## Pod Operations

```bash
# List pods
kubectl get pods
kubectl get pods -A              # all namespaces
kubectl get pods -o wide         # more details
kubectl get pods -w              # watch mode

# Create pod (imperative)
kubectl run nginx --image=nginx

# Create pod with resource limits
kubectl run nginx --image=nginx --requests='cpu=100m,memory=128Mi' --limits='cpu=200m,memory=256Mi'

# Delete pod
kubectl delete pod <pod-name>

# Get pod yaml
kubectl get pod <pod-name> -o yaml

# Describe pod
kubectl describe pod <pod-name>

# Pod logs
kubectl logs <pod-name>
kubectl logs -f <pod-name>           # follow
kubectl logs --previous <pod-name>   # previous container

# Execute in pod
kubectl exec -it <pod-name> -- sh
kubectl exec <pod-name> -- ls /app

# Port forward
kubectl port-forward <pod-name> 8080:80

# Copy files
kubectl cp <pod-name>:/path/file ./local-file
kubectl cp ./local-file <pod-name>:/path/file
```

## Labels & Selectors

```bash
# Add label
kubectl label pod <pod-name> env=prod

# Overwrite label
kubectl label pod <pod-name> env=staging --overwrite

# Remove label
kubectl label pod <pod-name> env-

# Show labels
kubectl get pods --show-labels

# Select by label
kubectl get pods -l app=frontend
kubectl get pods -l 'tier in (frontend,backend)'

# Custom columns
kubectl get pods -o custom-columns=NAME:.metadata.name,IMAGE:.spec.containers[0].image
```

## Resource Management

```bash
# Apply manifest
kubectl apply -f manifest.yaml

# Apply directory
kubectl apply -f ./configs/

# Delete from manifest
kubectl delete -f manifest.yaml

# Create from manifest
kubectl create -f manifest.yaml

# Replace (destructive)
kubectl replace -f manifest.yaml

# Edit live resource
kubectl edit pod <pod-name>

# Patch resource
kubectl patch pod <pod-name> -p '{"metadata":{"labels":{"env":"test"}}}'
```

## API Exploration

```bash
# List API resources
kubectl api-resources

# List API versions
kubectl api-versions

# Explain resource
kubectl explain pod
kubectl explain pod.spec
kubectl explain pod.spec.containers
```

## Output Formats

```bash
# YAML
kubectl get pod <pod-name> -o yaml

# JSON
kubectl get pod <pod-name> -o json

# JSONPath
kubectl get pods -o jsonpath='{.items[*].metadata.name}'

# Custom columns
kubectl get pods -o custom-columns=NAME:.metadata.name,STATUS:.status.phase

# Sort by field
kubectl get pods --sort-by=.metadata.creationTimestamp
```

## Quick Create Commands

```bash
# Create deployment
kubectl create deployment nginx --image=nginx --replicas=3

# Create service
kubectl expose deployment nginx --port=80 --target-port=80

# Create configmap
kubectl create configmap my-config --from-literal=key1=value1

# Create secret
kubectl create secret generic my-secret --from-literal=password=mypassword

# Quick dry-run (generate yaml)
kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > deployment.yaml
```

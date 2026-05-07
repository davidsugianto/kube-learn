# Kubernetes Fundamentals

Understanding the core architecture and concepts of Kubernetes.

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                      Control Plane                          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │
│  │  API Server │  │    etcd     │  │ Scheduler           │ │
│  │  (frontend) │  │  (storage)  │  │ (assigns pods)      │ │
│  └─────────────┘  └─────────────┘  └─────────────────────┘ │
│  ┌─────────────────────────────┐  ┌─────────────────────┐  │
│  │ Controller Manager          │  │ Cloud Controller    │  │
│  │ (maintains desired state)   │  │ (cloud integrations)│  │
│  └─────────────────────────────┘  └─────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                             │
         ┌───────────────────┼───────────────────┐
         ▼                   ▼                   ▼
┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
│     Node 1      │ │     Node 2      │ │     Node 3      │
│  ┌───────────┐  │ │  ┌───────────┐  │ │  ┌───────────┐  │
│  │  kubelet  │  │ │  │  kubelet  │  │ │  │  kubelet  │  │
│  └───────────┘  │ │  └───────────┘  │ │  └───────────┘  │
│  ┌───────────┐  │ │  ┌───────────┐  │ │  ┌───────────┐  │
│  │ kube-proxy│  │ │  │ kube-proxy│  │ │  │ kube-proxy│  │
│  └───────────┘  │ │  └───────────┘  │ │  └───────────┘  │
│  ┌───────────┐  │ │  ┌───────────┐  │ │  ┌───────────┐  │
│  │ Runtime   │  │ │  │ Runtime   │  │ │  │ Runtime   │  │
│  └───────────┘  │ │  └───────────┘  │ │  └───────────┘  │
│     Pods        │ │     Pods        │ │     Pods        │
└─────────────────┘ └─────────────────┘ └─────────────────┘
```

## Control Plane Components

### API Server
- The frontend for the Kubernetes control plane
- RESTful API for all operations
- Authentication, authorization, admission control
- Only component that talks to etcd directly

### etcd
- Distributed key-value store
- Stores all cluster data
- Consistent and highly available
- Always back up etcd!

### Scheduler
- Assigns pods to nodes
- Considers: resources, policies, affinity, taints
- Default scheduler can be customized

### Controller Manager
- Runs controller processes
- Maintains desired state
- Examples: Node Controller, Replication Controller, Endpoint Controller

### Cloud Controller Manager
- Cloud-specific control logic
- Integrates with cloud providers
- Node, Route, Service controllers

## Node Components

### kubelet
- Agent on each node
- Ensures containers run in pods
- Reports node status to API server
- Executes health checks

### kube-proxy
- Network proxy on each node
- Maintains network rules
- Implements Service abstraction
- Modes: iptables, IPVS, userspace

### Container Runtime
- Runs containers
- CRI-compatible runtimes: containerd, CRI-O
- Manages container lifecycle

## Core Concepts

### Cluster
A set of nodes (machines) that run containerized applications.

### Node
A worker machine (VM or physical) in the cluster.

```bash
# View nodes
kubectl get nodes -o wide

# Node details
kubectl describe node <node-name>
```

### Namespace
Virtual clusters within a physical cluster.

```bash
# List namespaces
kubectl get namespaces

# Create namespace
kubectl create namespace my-namespace

# Run in namespace
kubectl run nginx --image=nginx -n my-namespace
```

**Default namespaces:**
- `default` - For objects with no namespace
- `kube-system` - Kubernetes system components
- `kube-public` - Publicly accessible data
- `kube-node-lease` - Node heartbeats

### Pod
The smallest deployable unit - one or more containers.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.21
    ports:
    - containerPort: 80
```

### Labels & Selectors

**Labels** - Key-value pairs attached to objects.

```yaml
metadata:
  labels:
    app: frontend
    env: production
```

**Selectors** - Filter objects by labels.

```bash
# Equality-based
kubectl get pods -l app=frontend

# Set-based
kubectl get pods -l 'env in (production,staging)'
```

### Annotations
Non-identifying metadata - for tools, libraries, etc.

```yaml
metadata:
  annotations:
    example.com/build-id: "12345"
    description: "Frontend web server"
```

## Declarative vs Imperative

### Imperative (Commands)
```bash
kubectl run nginx --image=nginx
kubectl expose pod nginx --port=80
```

### Declarative (Manifests)
```bash
kubectl apply -f deployment.yaml
```

**Prefer declarative** for:
- Version control
- Reproducibility
- GitOps workflows
- Audit trails

## API Resources

```bash
# List all resource types
kubectl api-resources

# List API versions
kubectl api-versions

# Explain resource
kubectl explain pod
kubectl explain pod.spec
```

## Quick Reference

| Concept | Description |
|---------|-------------|
| Cluster | Set of nodes running K8s |
| Node | Worker machine |
| Pod | Smallest deployable unit |
| Namespace | Virtual cluster |
| Label | Key-value for identification |
| Selector | Filter by labels |
| Controller | Maintains desired state |

## Next Steps

Proceed to [02-workloads](../02-workloads) to learn about managing workloads.

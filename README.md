# Kubernetes Learning Path

A structured collection of notes, examples, and resources for learning Kubernetes — from basics to advanced topics.

## Learning Path

| Module | Topic | Focus Areas |
|--------|-------|-------------|
| [00-prerequisites](./00-prerequisites) | Prerequisites | Container basics, tools setup, local K8s |
| [01-fundamentals](./01-fundamentals) | Fundamentals | Architecture, core concepts, kubectl |
| [02-workloads](./02-workloads) | Workloads | Pods, Deployments, Jobs, DaemonSets |
| [03-networking](./03-networking) | Networking | Services, Ingress, NetworkPolicies |
| [04-storage](./04-storage) | Storage | PVs, PVCs, StorageClasses |
| [05-config-secrets](./05-config-secrets) | Config & Secrets | ConfigMaps, Secrets |
| [06-security](./06-security) | Security | RBAC, PodSecurity, ServiceAccounts |
| [07-observability](./07-observability) | Observability | Logging, monitoring, debugging |
| [08-scaling](./08-scaling) | Scaling | HPA, VPA, Cluster Autoscaler |
| [09-advanced](./09-advanced) | Advanced | Operators, CRDs, Service Mesh, GitOps |

## Quick Start

```bash
# Check prerequisites
kubectl version --client
minikube version  # or kind, k3d

# Run examples
kubectl apply -f examples/
```

## Resources

- [Official Kubernetes Docs](https://kubernetes.io/docs/)
- [Kubernetes API Reference](https://kubernetes.io/docs/reference/kubernetes-api/)
- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

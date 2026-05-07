# Observability - Commands

## Metrics

```bash
# Install metrics server
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# Node metrics
kubectl top nodes
kubectl top nodes --sort-by=memory

# Pod metrics
kubectl top pods
kubectl top pods -n <namespace>
kubectl top pods -A
kubectl top pods --sort-by=cpu

# Container metrics
kubectl top pod <pod-name> --containers

# Resource usage
kubectl top pods -l app=frontend
```

## Logs

```bash
# View pod logs
kubectl logs <pod-name>
kubectl logs <pod-name> -n <namespace>

# Follow logs
kubectl logs -f <pod-name>

# Tail logs
kubectl logs <pod-name> --tail=100

# Previous container logs
kubectl logs <pod-name> --previous

# Specific container
kubectl logs <pod-name> -c <container-name>

# All containers in pod
kubectl logs <pod-name> --all-containers=true

# Logs with timestamps
kubectl logs <pod-name> --timestamps

# Logs since time
kubectl logs <pod-name> --since=1h
kubectl logs <pod-name> --since-time=2024-01-01T00:00:00Z

# Logs from deployment
kubectl logs deployment/<deployment-name>

# Logs with label selector
kubectl logs -l app=frontend
kubectl logs -l app=frontend --max-log-requests=10

# Save logs to file
kubectl logs <pod-name> > pod.log
```

## Events

```bash
# Get events
kubectl get events
kubectl get events -n <namespace>
kubectl get events -A

# Sort events
kubectl get events --sort-by='.lastTimestamp'
kubectl get events --sort-by='.metadata.creationTimestamp'

# Watch events
kubectl get events -w

# Field selectors
kubectl get events --field-selector involvedObject.name=<pod-name>
kubectl get events --field-selector involvedObject.kind=Pod
kubectl get events --field-selector type=Warning

# Describe for events
kubectl describe pod <pod-name>
kubectl describe node <node-name>
```

## Debugging

```bash
# Describe resources
kubectl describe pod <pod-name>
kubectl describe node <node-name>
kubectl describe svc <service-name>
kubectl describe pvc <pvc-name>

# Debug pod
kubectl debug <pod-name> -it --image=busybox
kubectl debug <pod-name> -it --image=nicolaka/netshoot

# Debug node
kubectl debug node/<node-name> -it --image=busybox

# Debug with ephemeral container
kubectl debug <pod-name> -it --image=busybox --target=<container-name>

# Exec into container
kubectl exec -it <pod-name> -- sh
kubectl exec -it <pod-name> -c <container-name> -- sh

# Run command in container
kubectl exec <pod-name> -- ls /app
kubectl exec <pod-name> -- env

# Port forward for debugging
kubectl port-forward <pod-name> 8080:80
kubectl port-forward svc/<service-name> 8080:80
kubectl port-forward deployment/<deployment-name> 8080:80
```

## Common Debugging Scenarios

```bash
# Pod stuck in Pending
kubectl describe pod <pod-name>
kubectl get events --field-selector involvedObject.name=<pod-name>
kubectl get pvc  # Check if PVC is bound
kubectl get nodes  # Check node resources

# CrashLoopBackOff
kubectl logs <pod-name> --previous
kubectl describe pod <pod-name>

# ImagePullBackOff
kubectl describe pod <pod-name>
# Check: image name, imagePullSecrets, registry access

# OOMKilled
kubectl describe pod <pod-name>
kubectl top pods
kubectl get pod <pod-name> -o jsonpath='{.spec.containers[0].resources}'

# Not Ready
kubectl describe pod <pod-name>
kubectl logs <pod-name>
# Check readiness probe

# Service not working
kubectl get endpoints <service-name>
kubectl describe svc <service-name>
kubectl exec -it <pod-name> -- nslookup <service-name>
```

## Network Debugging

```bash
# Test DNS
kubectl run -it --rm debug --image=busybox -- nslookup kubernetes
kubectl run -it --rm debug --image=busybox -- nslookup <service-name>

# Test connectivity
kubectl run -it --rm debug --image=busybox -- wget -qO- http://<service-name>:80
kubectl run -it --rm debug --image=curlimages/curl -- curl http://<service-name>

# Check service endpoints
kubectl get endpoints <service-name>
kubectl describe endpoints <service-name>

# Test from specific namespace
kubectl run -it --rm debug -n <namespace> --image=busybox -- wget -qO- http://<service-name>:80
```

## Resource Analysis

```bash
# Get pod resource requests/limits
kubectl get pods -o custom-columns=NAME:.metadata.name,CPU_REQ:.spec.containers[*].resources.requests.cpu,MEM_REQ:.spec.containers[*].resources.requests.memory

# Get resource usage vs limits
kubectl top pods
kubectl get pods -o custom-columns=NAME:.metadata.name,CPU_LIMIT:.spec.containers[*].resources.limits.cpu,MEM_LIMIT:.spec.containers[*].resources.limits.memory

# Find pods without resource limits
kubectl get pods -A -o jsonpath='{range .items[?(!.spec.containers[*].resources.limits)]}{.metadata.name}{"\n"}{end}'

# Find pods using most resources
kubectl top pods --sort-by=cpu
kubectl top pods --sort-by=memory
```

## Cluster Information

```bash
# Cluster info
kubectl cluster-info
kubectl cluster-info dump

# Node information
kubectl get nodes -o wide
kubectl describe node <node-name>

# Node conditions
kubectl get nodes -o custom-columns=NAME:.metadata.name,STATUS:.status.conditions[?(@.type=="Ready")].status

# Component statuses
kubectl get cs  # Deprecated in newer versions
kubectl get --raw='/readyz?verbose'

# API health
kubectl get --raw='/healthz'
kubectl get --raw='/livez'
kubectl get --raw='/readyz'
```

## Quick Debug Pods

```bash
# Busybox
kubectl run -it --rm debug --image=busybox -- sh

# Netshoot (network debugging)
kubectl run -it --rm debug --image=nicolaka/netshoot -- bash

# Curl
kubectl run -it --rm debug --image=curlimages/curl -- sh

# Ubuntu
kubectl run -it --rm debug --image=ubuntu -- bash
```

# Scaling - Commands

## HPA Operations

```bash
# Create HPA
kubectl autoscale deployment nginx --min=2 --max=10 --cpu-percent=80

# Get HPAs
kubectl get horizontalpodautoscalers
kubectl get hpa

# Get HPA details
kubectl describe hpa <hpa-name>

# Get HPA YAML
kubectl get hpa <hpa-name> -o yaml

# Edit HPA
kubectl edit hpa <hpa-name>

# Delete HPA
kubectl delete hpa <hpa-name>

# Watch HPA status
kubectl get hpa -w
```

## VPA Operations

```bash
# Get VPAs
kubectl get verticalpodautoscalers
kubectl get vpa

# Get VPA details
kubectl describe vpa <vpa-name>

# View VPA recommendations
kubectl get vpa <vpa-name> -o jsonpath='{.status.recommendation.containerRecommendations}'

# Get VPA YAML
kubectl get vpa <vpa-name> -o yaml

# Delete VPA
kubectl delete vpa <vpa-name>
```

## Cluster Autoscaler

```bash
# Check cluster autoscaler logs
kubectl logs -n kube-system -l app=cluster-autoscaler

# Get cluster autoscaler pods
kubectl get pods -n kube-system -l app=cluster-autoscaler

# View nodes
kubectl get nodes

# Cordon node (prevent new pods)
kubectl cordon <node-name>

# Uncordon node
kubectl uncordon <node-name>

# Drain node
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data
```

## Resource Management

```bash
# Get resource quotas
kubectl get resourcequotas
kubectl get quota

# Describe resource quota
kubectl describe quota <quota-name>

# Get limit ranges
kubectl get limitranges
kubectl get limits

# Describe limit range
kubectl describe limits <limit-name>

# View namespace resource usage
kubectl describe namespace <namespace>
```

## Manual Scaling

```bash
# Scale deployment
kubectl scale deployment nginx --replicas=5

# Scale replicaset
kubectl scale rs <rs-name> --replicas=5

# Scale statefulset
kubectl scale statefulset web --replicas=3

# Conditional scaling (only if current is 3)
kubectl scale deployment nginx --current-replicas=3 --replicas=5
```

## Resource Analysis

```bash
# View pod resources
kubectl top pods
kubectl top pods --sort-by=cpu
kubectl top pods --sort-by=memory

# View node resources
kubectl top nodes

# Get resource requests/limits
kubectl get pods -o custom-columns=NAME:.metadata.name,CPU:.spec.containers[*].resources.requests.cpu,MEM:.spec.containers[*].resources.requests.memory

# Find pods without limits
kubectl get pods -A -o jsonpath='{range .items[?(!.spec.containers[*].resources.limits)]}{.metadata.name}{"\n"}{end}'

# Calculate total resources in namespace
kubectl get pods -o jsonpath='{range .items[*].spec.containers[*]}{.resources.requests.cpu}{"\n"}{end}'
```

## KEDA Operations

```bash
# Install KEDA
kubectl apply -f https://github.com/kedacore/keda-2operator/releases/download/v2.10.0/keda-2.10.0.yaml

# Get ScaledObjects
kubectl get scaledobjects
kubectl get so

# Get ScaledJobs
kubectl get scaledjobs

# Get TriggerAuthentication
kubectl get triggerauthentications

# View KEDA logs
kubectl logs -n keda-system -l app=keda-operator

# Delete ScaledObject
kubectl delete scaledobject <name>
```

## Testing Scaling

```bash
# Generate load (for HPA testing)
kubectl run -i --tty load-generator --rm --image=busybox --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://nginx; done"

# Watch HPA
kubectl get hpa -w

# Watch pods
kubectl get pods -w

# Monitor scaling events
kubectl get events --field-selector reason=Scaled
```

## Quick YAML Generation

```bash
# Generate HPA YAML
kubectl autoscale deployment nginx --min=2 --max=10 --cpu-percent=80 --dry-run=client -o yaml > hpa.yaml

# Generate ResourceQuota YAML
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
spec:
  hard:
    requests.cpu: "10"
    requests.memory: 20Gi
    limits.cpu: "20"
    limits.memory: 40Gi
EOF
```

## Useful Queries

```bash
# All HPAs in cluster
kubectl get hpa -A

# HPAs with target metrics
kubectl get hpa -A -o custom-columns=NAME:.metadata.name,TARGET:.spec.scaleTargetRef.name,MIN:.spec.minReplicas,MAX:.spec.maxReplicas,REPLICAS:.status.currentReplicas

# Nodes with capacity
kubectl get nodes -o custom-columns=NAME:.metadata.name,CPU:.status.capacity.cpu,MEM:.status.capacity.memory

# Pod distribution across nodes
kubectl get pods -A -o jsonpath='{range .items[*]}{.spec.nodeName}{"\n"}{end}' | sort | uniq -c
```

# Kubernetes Certification Prep - Interview Questions

## CKA (Certified Kubernetes Administrator) Style Questions

### 1. Create a pod with specific resource requirements

**Task:** Create a pod named `resource-pod` with:
- Image: nginx
- CPU request: 100m
- Memory request: 128Mi
- CPU limit: 200m
- Memory limit: 256Mi

**Answer:**
```bash
# Imperative command
kubectl run resource-pod --image=nginx --requests='cpu=100m,memory=128Mi' --limits='cpu=200m,memory=256Mi'

# Or create YAML
kubectl run resource-pod --image=nginx --dry-run=client -o yaml > pod.yaml
# Edit to add resources
kubectl apply -f pod.yaml
```

### 2. Create a deployment and scale it

**Task:**
1. Create a deployment named `nginx-deploy` with 3 replicas using nginx:1.19 image
2. Scale it to 5 replicas
3. Update the image to nginx:1.20

**Answer:**
```bash
# Create deployment
kubectl create deployment nginx-deploy --image=nginx:1.19 --replicas=3

# Scale to 5 replicas
kubectl scale deployment nginx-deploy --replicas=5

# Update image
kubectl set image deployment/nginx-deploy nginx=nginx:1.20
```

### 3. Configure a node to be unschedulable

**Task:** Make node `node-1` unschedulable and evict all running pods.

**Answer:**
```bash
# Cordon the node (prevent new pods)
kubectl cordon node-1

# Drain the node (evict existing pods)
kubectl drain node-1 --ignore-daemonsets --delete-emptydir-data --force
```

## CKAD (Certified Kubernetes Application Developer) Style Questions

### 4. Create a ConfigMap and use it in a pod

**Task:**
1. Create a ConfigMap named `app-config` with key `DB_HOST` and value `postgres`
2. Create a pod that uses this ConfigMap as an environment variable

**Answer:**
```bash
# Create ConfigMap
kubectl create configmap app-config --from-literal=DB_HOST=postgres

# Create pod with ConfigMap
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: config-pod
spec:
  containers:
  - name: app
    image: nginx
    env:
    - name: DB_HOST
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: DB_HOST
EOF
```

### 5. Create a multi-container pod

**Task:** Create a pod named `multi-pod` with:
- Container 1: nginx on port 80
- Container 2: busybox that writes to a shared volume

**Answer:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-pod
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
    volumeMounts:
    - name: shared-data
      mountPath: /usr/share/nginx/html
  - name: busybox
    image: busybox
    command: ['sh', '-c', 'while true; do echo $(date) >> /data/index.html; sleep 5; done']
    volumeMounts:
    - name: shared-data
      mountPath: /data
  volumes:
  - name: shared-data
    emptyDir: {}
```

## CKS (Certified Kubernetes Security Specialist) Style Questions

### 6. Create a Role and RoleBinding

**Task:**
1. Create a Role named `pod-reader` that can get, list, and watch pods
2. Create a RoleBinding named `read-pods` that binds the role to user `jane`

**Answer:**
```yaml
# Role
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
---
# RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
subjects:
- kind: User
  name: jane
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

### 7. Create a NetworkPolicy

**Task:** Create a NetworkPolicy that denies all ingress traffic to pods with label `app=secure`.

**Answer:**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-ingress
spec:
  podSelector:
    matchLabels:
      app: secure
  policyTypes:
  - Ingress
```

## Time Management Tips

### CKA Time Management
- **Total time:** 2 hours
- **Questions:** 15-17
- **Average per question:** 7-8 minutes

### CKAD Time Management
- **Total time:** 2 hours
- **Questions:** 19
- **Average per question:** 6 minutes

### CKS Time Management
- **Total time:** 2 hours
- **Questions:** 15-17
- **Average per question:** 7-8 minutes

## Quick Reference Commands

```bash
# Generate YAML quickly
kubectl run pod --image=nginx --dry-run=client -o yaml > pod.yaml
kubectl create deploy deploy --image=nginx --dry-run=client -o yaml > deploy.yaml

# Expose services
kubectl expose pod pod --port=80 --target-port=8080

# Rollout
kubectl rollout status deploy/deploy
kubectl rollout undo deploy/deploy

# Resource management
kubectl top nodes
kubectl top pods

# RBAC
kubectl auth can-i get pods --as=jane

# Debugging
kubectl exec -it pod -- sh
kubectl port-forward pod 8080:80
```

## Practice Tips

1. **Use kubectl --help extensively**
2. **Practice imperative commands**
3. **Know how to use --dry-run=client -o yaml**
4. **Bookmark questions you're unsure about**
5. **Verify your work**
6. **Use aliases** (k=kubectl)
7. **Practice with time constraints**
8. **Review official documentation** (allowed during exam)

## Common Exam Patterns

1. **Troubleshooting:** Fix broken pods/deployments
2. **Configuration:** Create ConfigMaps, Secrets
3. **Networking:** Services, Ingress, NetworkPolicies
4. **Storage:** PV, PVC, StorageClass
5. **Security:** RBAC, Security Contexts
6. **Scaling:** HPA, manual scaling
7. **Rollouts:** Deployments, rollbacks
8. **Scheduling:** Node selectors, taints, tolerations

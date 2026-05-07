# Security

Comprehensive security practices for Kubernetes clusters and workloads.

## RBAC (Role-Based Access Control)

Controls who can access what resources in the cluster.

### Key Concepts

| Concept | Scope | Description |
|---------|-------|-------------|
| Role | Namespace | Rules for namespace resources |
| ClusterRole | Cluster-wide | Rules for cluster resources |
| RoleBinding | Namespace | Binds Role to subjects |
| ClusterRoleBinding | Cluster-wide | Binds ClusterRole to subjects |

### Role

Defines permissions within a namespace.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: default
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
- apiGroups: [""]
  resources: ["pods/log"]
  verbs: ["get"]
```

### ClusterRole

Defines cluster-wide permissions.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: node-reader
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["get", "list", "watch"]
```

### RoleBinding

Binds a Role to subjects (users, groups, service accounts).

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: User
  name: jane
  apiGroup: rbac.authorization.k8s.io
- kind: ServiceAccount
  name: my-service-account
  namespace: default
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

### ClusterRoleBinding

Binds a ClusterRole to subjects.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: read-nodes
subjects:
- kind: User
  name: jane
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: node-reader
  apiGroup: rbac.authorization.k8s.io
```

## ServiceAccount

Identity for processes running in pods.

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-service-account
  namespace: default
```

### Using ServiceAccount in Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  serviceAccountName: my-service-account
  automountServiceAccountToken: true  # Default: true
  containers:
  - name: app
    image: myapp
```

### Disable Token Automount

```yaml
spec:
  automountServiceAccountToken: false
```

## Pod Security Standards

Three policy levels:

| Level | Description |
|-------|-------------|
| **Privileged** | Unrestricted policy, widest permission |
| **Baseline** | Minimally restrictive, prevents known privilege escalations |
| **Restricted** | Heavily restricted, follows hardening best practices |

### Pod Security Admission

Namespace labels for enforcement:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: my-namespace
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/enforce-version: latest
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/warn: restricted
```

### Restricted Pod Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: restricted-pod
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    fsGroup: 1000
    seccompProfile:
      type: RuntimeDefault
  containers:
  - name: app
    image: nginx
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop:
        - ALL
```

## Security Context

Security settings at pod and container level.

### Pod Security Context

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
    runAsNonRoot: true
    seccompProfile:
      type: RuntimeDefault
```

### Container Security Context

```yaml
spec:
  containers:
  - name: app
    image: nginx
    securityContext:
      runAsUser: 1000
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        add: ["NET_ADMIN"]
        drop: ["ALL"]
      privileged: false
```

## Network Policies

Control pod-to-pod communication (see 03-networking).

## Node Restriction

Limit what nodes a pod can run on.

### Node Selector

```yaml
spec:
  nodeSelector:
    disktype: ssd
```

### Node Affinity

```yaml
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: zone
            operator: In
            values:
            - us-west-1
```

### Taints and Tolerations

**Taint nodes:**
```bash
kubectl taint nodes node1 key=value:NoSchedule
kubectl taint nodes node1 key=value:NoExecute
kubectl taint nodes node1 key=value:PreferNoSchedule
```

**Tolerations:**
```yaml
spec:
  tolerations:
  - key: "key"
    operator: "Equal"
    value: "value"
    effect: "NoSchedule"
```

## Image Security

### Private Registry

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: registry-secret
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: <base64-encoded-docker-config>
---
apiVersion: v1
kind: Pod
metadata:
  name: private-pod
spec:
  imagePullSecrets:
  - name: registry-secret
  containers:
  - name: app
    image: private-registry.io/myapp:v1
```

### Image Policy

Use Admission Controllers to enforce image policies:
- ImagePolicyWebhook
- Check image signatures
- Require approved registries

## Audit Logging

Track requests to the API server.

### Audit Policy

```yaml
apiVersion: audit.k8s.io/v1
kind: Policy
rules:
- level: Metadata
  resources:
  - group: ""
    resources: ["secrets"]
- level: RequestResponse
  resources:
  - group: ""
    resources: ["pods"]
```

## Encryption at Rest

Encrypt secrets in etcd.

### Encryption Configuration

```yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
    - secrets
    providers:
    - aescbc:
        keys:
        - name: key1
          secret: <base64-encoded-secret>
    - identity: {}
```

## Security Checklist

- [ ] Enable RBAC
- [ ] Use Pod Security Standards
- [ ] Configure Network Policies
- [ ] Encrypt secrets at rest
- [ ] Use Service Accounts with minimal permissions
- [ ] Enable audit logging
- [ ] Run containers as non-root
- [ ] Use read-only root filesystem
- [ ] Drop unnecessary capabilities
- [ ] Use approved base images
- [ ] Scan images for vulnerabilities
- [ ] Keep Kubernetes updated

## Next Steps

Proceed to [07-observability](../07-observability) to learn about monitoring and debugging.

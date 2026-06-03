# Kubernetes Security - Interview Questions

## Beginner Questions

### 1. What is RBAC?

**Answer:**
RBAC (Role-Based Access Control) regulates access based on roles within an organization.

**Components:**
- **Role**: Rules within a namespace
- **ClusterRole**: Cluster-wide rules
- **RoleBinding**: Binds Role to subjects
- **ClusterRoleBinding**: Binds ClusterRole to subjects

### 2. What is a ServiceAccount?

**Answer:**
A ServiceAccount provides an identity for processes running in a Pod.

**Default:** Every namespace has a default ServiceAccount.

**Use cases:**
- Pod-to-API server authentication
- Granting permissions to applications
- CI/CD automation

### 3. What is the difference between Role and ClusterRole?

**Answer:**

| Aspect | Role | ClusterRole |
|--------|------|-------------|
| Scope | Namespace | Cluster-wide |
| Resources | Namespace-scoped | Any resource |
| Use case | Application access | Cluster administration |

### 4. What are Secrets in Kubernetes?

**Answer:**
Secrets store sensitive data (passwords, tokens, keys).

**Types:**
- `Opaque`: Arbitrary user data (default)
- `kubernetes.io/tls`: TLS certificates
- `kubernetes.io/dockerconfigjson`: Docker registry
- `kubernetes.io/service-account-token`: Service account token

### 5. What is a Security Context?

**Answer:**
Security Context defines privilege and access control settings for Pods/containers.

**Settings:**
- Run as specific user/group
- Run as non-root
- Privileged mode
- Capabilities (add/drop)
- Read-only root filesystem

## Intermediate Questions

### 6. How do you create a Role and RoleBinding?

**Answer:**
```yaml
# Role
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: default
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
  namespace: default
subjects:
- kind: User
  name: jane
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

### 7. What are Pod Security Standards?

**Answer:**
Three policy levels:

| Level | Description |
|-------|-------------|
| **Privileged** | Unrestricted, widest permission |
| **Baseline** | Minimally restrictive, prevents known escalations |
| **Restricted** | Heavily restricted, hardening best practices |

**Enforcement:** Via Pod Security Admission or 3rd party (OPA, Kyverno)

### 8. What is the difference between ConfigMap and Secret?

**Answer:**

| Aspect | ConfigMap | Secret |
|--------|-----------|--------|
| Purpose | Non-sensitive data | Sensitive data |
| Encoding | Plain text | Base64 |
| Encryption | No | Can be encrypted at rest |
| Size limit | 1 MiB | 1 MiB |

### 9. What is mTLS in Kubernetes?

**Answer:**
Mutual TLS (mTLS) provides bidirectional authentication.

**In Kubernetes:**
- Service mesh (Istio, Linkerd) provides automatic mTLS
- Encrypts traffic between services
- Authenticates both client and server

### 10. How do you encrypt Secrets at rest?

**Answer:**
**Step 1: Create encryption key**
```bash
head -c 32 /dev/urandom | base64
```

**Step 2: Create EncryptionConfiguration**
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
          secret: <base64-encoded-key>
    - identity: {}
```

**Step 3: Configure API server**
```bash
--encryption-provider-config=/etc/kubernetes/encryption-config.yaml
```

## Advanced Questions

### 11. What is the principle of least privilege?

**Answer:**
Grant only the permissions needed to perform required tasks.

**Implementation:**
1. Use namespace-scoped Roles when possible
2. Limit resource access
3. Use specific verbs (not "*")
4. Regularly audit RBAC rules
5. Use service accounts with minimal permissions

### 12. What is admission control?

**Answer:**
Admission controllers intercept requests before persistence.

**Types:**
- **Validating**: Check validity
- **Mutating**: Modify requests

**Examples:**
- ResourceQuota
- LimitRanger
- PodSecurity
- NodeRestriction
- Custom webhooks

### 13. What is OPA Gatekeeper?

**Answer:**
OPA (Open Policy Agent) Gatekeeper is a policy engine for Kubernetes.

**Use cases:**
- Enforce custom policies
- Validate resource creation
- Audit existing resources

**Example policy:**
```yaml
apiVersion: templates.gatekeeper.sh/v1
kind: ConstraintTemplate
metadata:
  name: k8srequiredlabels
spec:
  crd:
    spec:
      names:
        kind: K8sRequiredLabels
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8srequiredlabels
        violation[{"msg": msg}] {
          provided := {label | input.review.object.metadata.labels[label]}
          required := {label | label := input.parameters.labels[_]}
          missing := required - provided
          count(missing) > 0
          msg := sprintf("Missing required labels: %v", [missing])
        }
```

### 14. How do you secure the Kubernetes API server?

**Answer:**
**1. Authentication:**
- Use OIDC for user authentication
- Certificate-based auth for services
- Disable anonymous access

**2. Authorization:**
- Enable RBAC
- Disable ABAC
- Use Node authorization

**3. Admission Control:**
- Enable admission controllers
- Use custom policies (OPA, Kyverno)

**4. Network Security:**
- Restrict API server access
- Use firewall rules
- Enable audit logging

**5. Encryption:**
- Encrypt etcd at rest
- Use TLS for all communication

### 15. What is supply chain security?

**Answer:**
Protecting the software supply chain from source to deployment.

**Key aspects:**
1. **Image security:**
   - Scan for vulnerabilities
   - Sign images
   - Use trusted registries

2. **Dependency management:**
   - Audit dependencies
   - Update regularly

3. **Build security:**
   - Secure CI/CD pipelines
   - Immutable builds

4. **Deployment security:**
   - Verify image signatures
   - Admission controllers

**Tools:**
- Trivy (scanning)
- Cosign (signing)
- Sigstore (supply chain security)

### 16. How do you implement network security?

**Answer:**
**1. Network Policies:**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-ingress
spec:
  podSelector: {}
  policyTypes:
  - Ingress
```

**2. Service Mesh:**
- mTLS between services
- Traffic policies
- Authorization policies

**3. Ingress Security:**
- TLS termination
- Rate limiting
- WAF (Web Application Firewall)

**4. CNI Security:**
- Choose secure CNI
- Enable network policies

## Scenario Questions

### 17. How would you secure a production cluster?

**Answer:**
**Control Plane:**
- Enable RBAC
- Encrypt Secrets at rest
- Enable audit logging
- Restrict API server access
- Use secure etcd configuration

**Workloads:**
- Pod Security Standards (Restricted)
- Resource limits and quotas
- Security contexts
- Network policies
- Image scanning

**Access:**
- OIDC authentication
- Least privilege RBAC
- Service accounts with minimal permissions
- Regular access audits

**Network:**
- Network policies
- Service mesh with mTLS
- Ingress with TLS
- Private networking

**Monitoring:**
- Audit logs
- Security alerts
- Vulnerability scanning
- Intrusion detection

### 18. How do you rotate Secrets?

**Answer:**
**Method 1: Manual rotation**
```bash
# Update Secret
kubectl create secret generic my-secret --from-literal=password=newpassword --dry-run=client -o yaml | kubectl apply -f -

# Restart Pods
kubectl rollout restart deployment/<deployment-name>
```

**Method 2: External Secrets Operator**
```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: my-secret
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: vault-backend
  target:
    name: my-secret
  data:
  - secretKey: password
    remoteRef:
      key: /myapp/password
```

**Best practices:**
- Regular rotation schedule
- Automated rotation
- Zero-downtime rotation
- Audit rotation events

### 19. A Pod needs to access the Kubernetes API. How do you secure it?

**Answer:**
**1. Create ServiceAccount:**
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-app-sa
```

**2. Create Role with minimal permissions:**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
```

**3. Create RoleBinding:**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: my-app-pod-reader
subjects:
- kind: ServiceAccount
  name: my-app-sa
roleRef:
  kind: Role
  name: pod-reader
```

**4. Use ServiceAccount in Pod:**
```yaml
spec:
  serviceAccountName: my-app-sa
```

### 20. How do you secure container images?

**Answer:**
**1. Use minimal base images:**
```dockerfile
FROM alpine:latest  # or distroless
```

**2. Don't run as root:**
```dockerfile
USER 1000:1000
```

**3. Scan images:**
```bash
trivy image myimage:latest
```

**4. Sign images:**
```bash
cosign sign myimage:latest
```

**5. Verify signatures:**
```bash
cosign verify myimage:latest
```

**6. Use admission controllers:**
- Verify image signatures
- Block vulnerable images
- Enforce approved registries

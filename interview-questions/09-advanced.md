# Advanced Kubernetes Topics - Interview Questions

## Operators and CRDs

### 1. What is a Custom Resource Definition (CRD)?

**Answer:**
A CRD extends the Kubernetes API with custom resources.

**Example:**
```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: crontabs.stable.example.com
spec:
  group: stable.example.com
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              cronSpec:
                type: string
  scope: Namespaced
  names:
    plural: crontabs
    singular: crontab
    kind: CronTab
```

### 2. What is an Operator?

**Answer:**
An Operator is a method of packaging, deploying, and managing a Kubernetes application.

**Components:**
- Custom Resource Definition (CRD)
- Controller
- Application-specific knowledge

**Use cases:**
- Databases (PostgreSQL, MySQL)
- Message queues (Kafka)
- Monitoring (Prometheus)

### 3. What is the Operator pattern?

**Answer:**
The Operator pattern captures human operational knowledge in software.

**Controller loop:**
```
while true:
    desired_state = get_custom_resource()
    current_state = get_actual_state()
    if desired_state != current_state:
        reconcile()
```

**Benefits:**
- Automation
- Consistency
- Error handling
- Self-healing

### 4. How do you create an Operator?

**Answer:**
**Tools:**
1. **Operator SDK**: Go, Ansible, Helm
2. **Kubebuilder**: Go framework
3. **KUDO**: Declarative operators

**Steps with Operator SDK:**
```bash
# Initialize
operator-sdk init --domain my.domain --repo github.com/my-domain/my-operator

# Create API
operator-sdk create api --group cache --version v1alpha1 --kind Memcached --resource --controller

# Implement reconciliation logic
# Build and push image
make docker-build docker-push IMG=myrepo/my-operator:v1

# Deploy
make deploy IMG=myrepo/my-operator:v1
```

## Service Mesh

### 5. What is a Service Mesh?

**Answer:**
A Service Mesh provides infrastructure for service-to-service communication.

**Features:**
- Traffic management
- Security (mTLS)
- Observability
- Policy enforcement

**Popular options:**
- Istio
- Linkerd
- Consul Connect
- Cilium

### 6. How does Istio work?

**Answer:**
**Components:**
- **Envoy**: Sidecar proxy
- **Istiod**: Control plane (Pilot, Citadel, Galley)

**Traffic flow:**
```
Service A → Envoy A → Envoy B → Service B
              ↓          ↓
           mTLS      mTLS
```

**Key features:**
- Traffic splitting
- Circuit breaking
- Retries and timeouts
- Fault injection
- mTLS encryption

### 7. What is mTLS and why is it important?

**Answer:**
mTLS (Mutual TLS) provides bidirectional authentication.

**Benefits:**
- Service-to-service authentication
- Encrypted communication
- Certificate management
- Zero-trust security

**Implementation:**
```yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: istio-system
spec:
  mtls:
    mode: STRICT
```

## GitOps

### 8. What is GitOps?

**Answer:**
GitOps is a paradigm for Kubernetes cluster management using Git as the source of truth.

**Principles:**
1. Declarative configuration
2. Git as single source of truth
3. Automated application
4. Continuous reconciliation

**Tools:**
- ArgoCD
- Flux
- Jenkins X

### 9. How does ArgoCD work?

**Answer:**
ArgoCD is a declarative GitOps continuous delivery tool.

**Workflow:**
1. Git repository contains manifests
2. ArgoCD watches repository
3. Compares Git state with cluster state
4. Syncs when out of sync

**Key features:**
- Automated sync
- Rollback
- Health status
- Multi-cluster

### 10. What is the difference between ArgoCD and Flux?

**Answer:**

| Feature | ArgoCD | Flux |
|---------|--------|------|
| UI | Web UI + CLI | CLI only |
| Architecture | Monolithic | Modular |
| Rollback | Native | Manual |
| Multi-cluster | Native | Via Flux instances |
| Helm | Native | Via Helm controller |

**Choose ArgoCD for:**
- Visual interface
- Easy rollback
- Multi-cluster management

**Choose Flux for:**
- Lightweight setup
- GitOps toolkit approach
- CNCF project

## Multi-Cluster

### 11. What is multi-cluster Kubernetes?

**Answer:**
Running multiple Kubernetes clusters for:
- High availability
- Geographic distribution
- Scaling
- Isolation
- Cost optimization

**Patterns:**
- Federation
- Multi-cluster services
- Cluster API

### 12. What is Cluster API?

**Answer:**
Cluster API provides declarative cluster management.

**Components:**
- **Cluster**: Cluster definition
- **Machine**: Node definition
- **MachineDeployment**: Node group
- **MachineSet**: Node replica set

**Benefits:**
- Declarative cluster creation
- Multi-cloud support
- Automated upgrades
- GitOps-friendly

### 13. How do you implement multi-cluster service discovery?

**Answer:**
**Approaches:**

**1. Service mirrors:**
```yaml
apiVersion: multicluster.x-k8s.io/v1alpha1
kind: ServiceExport
metadata:
  name: my-service
```

**2. DNS-based:**
- CoreDNS federation
- Multi-cluster DNS

**3. Service mesh:**
- Istio multi-cluster
- Linkerd multi-cluster

**4. Application-level:**
- External-dns
- Custom DNS

## Advanced Security

### 14. What is supply chain security?

**Answer:**
Securing the software supply chain from source to deployment.

**Components:**
1. **Image security:**
   - Vulnerability scanning
   - Image signing
   - Base image management

2. **Build security:**
   - Secure CI/CD
   - Reproducible builds
   - Build attestations

3. **Deployment security:**
   - Image verification
   - Admission controllers
   - Runtime security

**Tools:**
- Sigstore
- Trivy
- Cosign
- Snyk

### 15. What is image signing and why is it important?

**Answer:**
Image signing provides cryptographic proof of image authenticity.

**Benefits:**
- Verify image integrity
- Prove image origin
- Prevent tampering
- Compliance requirements

**Implementation with Cosign:**
```bash
# Generate key
cosign generate-key-pair

# Sign image
cosign sign --key cosign.key myimage:v1

# Verify signature
cosign verify --key cosign.pub myimage:v1
```

**Policy enforcement:**
```yaml
apiVersion: policy.sigstore.dev/v1beta1
kind: ClusterImagePolicy
metadata:
  name: my-policy
spec:
  images:
  - glob: "myrepo.io/*"
  authorities:
  - key:
      data: |
        -----BEGIN PUBLIC KEY-----
        ...
        -----END PUBLIC KEY-----
```

## Advanced Networking

### 16. What is Cilium and how does it differ from traditional CNI?

**Answer:**
Cilium is an eBPF-based networking and security solution.

**Differences:**
| Aspect | Traditional CNI | Cilium |
|--------|----------------|--------|
| Implementation | iptables, IPVS | eBPF |
| Performance | Good | Better |
| Visibility | Limited | Deep |
| Security | Basic | Advanced |
| Features | Networking | Network + Security + Observability |

**Features:**
- Layer 3-7 policies
- Deep visibility
- High performance
- Service mesh (no sidecar)

### 17. How do you implement canary deployments with a service mesh?

**Answer:**
**Istio VirtualService:**
```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: my-app
spec:
  hosts:
  - my-app
  http:
  - route:
    - destination:
        host: my-app
        subset: stable
      weight: 90
    - destination:
        host: my-app
        subset: canary
      weight: 10
```

**Progressive rollout:**
1. Start with 10% traffic to canary
2. Monitor metrics
3. Gradually increase traffic
4. Complete rollout or rollback

## Performance and Scaling

### 18. How do you optimize Kubernetes performance?

**Answer:**
**Cluster level:**
- Right-size nodes
- Use cluster autoscaler
- Optimize etcd
- Use IPVS proxy mode

**Application level:**
- Set appropriate resource limits
- Use HPA/VPA
- Optimize container images
- Use caching

**Networking:**
- Choose appropriate CNI
- Use service mesh efficiently
- Optimize DNS

**Storage:**
- Use appropriate StorageClass
- Consider local storage for performance
- Optimize disk I/O

### 19. What are the limits of Kubernetes scaling?

**Answer:**
**Official limits (v1.28):**
- 5,000 nodes per cluster
- 150,000 total Pods
- 300,000 total containers
- 100 Pods per node

**Bottlenecks:**
- etcd performance
- API server load
- Controller manager
- Scheduler

**Optimizations:**
- Split large clusters
- Use cluster federation
- Optimize queries
- Use caching

### 20. How do you implement platform engineering with Kubernetes?

**Answer:**
**Components:**
1. **Internal Developer Platform (IDP):**
   - Self-service provisioning
   - Templates and blueprints
   - Documentation

2. **Infrastructure as Code:**
   - Terraform, Pulumi
   - GitOps workflows

3. **Developer experience:**
   - CLI tools
   - Web portals
   - Documentation

4. **Abstractions:**
   - Custom resources
   - Operators
   - Helm charts

**Tools:**
- Backstage
- Crossplane
- Kratix
- Humanitec

**Benefits:**
- Faster development
- Consistency
- Reduced cognitive load
- Better governance

# Kubernetes Basics - Interview Questions

## Beginner Questions

### 1. What is Kubernetes?

**Answer:**
Kubernetes (K8s) is an open-source container orchestration platform that automates the deployment, scaling, and management of containerized applications. It was originally developed by Google and is now maintained by the Cloud Native Computing Foundation (CNCF).

**Key features:**
- Service discovery and load balancing
- Storage orchestration
- Automated rollouts and rollbacks
- Self-healing
- Secret and configuration management
- Automatic bin packing

### 2. What is the difference between Docker and Kubernetes?

**Answer:**
- **Docker**: Container runtime that packages and runs containers on a single host
- **Kubernetes**: Orchestration platform that manages containers across multiple hosts

| Aspect | Docker | Kubernetes |
|--------|--------|------------|
| Scope | Single host | Multiple hosts |
| Scaling | Manual | Automatic |
| Load Balancing | Manual | Built-in |
| Self-healing | No | Yes |
| Service Discovery | Limited | Built-in |

### 3. What is a Pod?

**Answer:**
A Pod is the smallest deployable unit in Kubernetes. It represents a single instance of a running process in the cluster and can contain one or more containers that share:
- Network namespace (same IP address)
- Storage volumes
- Inter-process communication

**Key points:**
- Pods are ephemeral (can be destroyed and recreated)
- Each Pod gets a unique IP address
- Containers in a Pod share localhost
- Pods are rarely created directly; use controllers (Deployments, etc.)

### 4. What is a Node?

**Answer:**
A Node is a worker machine in Kubernetes (can be a VM or physical machine). Each node runs:
- **kubelet**: Agent that communicates with the control plane
- **kube-proxy**: Network proxy for service abstraction
- **Container runtime**: Software to run containers (containerd, CRI-O)

**Types:**
- Worker nodes: Run application workloads
- Control plane nodes: Run cluster management components

### 5. What is a Namespace?

**Answer:**
Namespaces provide a mechanism for isolating groups of resources within a single cluster. They provide:
- Resource isolation (objects in different namespaces are separate)
- Resource quotas (limit resource usage per namespace)
- Access control (RBAC policies can be namespace-scoped)

**Default namespaces:**
- `default`: For objects with no namespace specified
- `kube-system`: Kubernetes system components
- `kube-public`: Publicly accessible data
- `kube-node-lease`: Node heartbeat data

## Intermediate Questions

### 6. What is the difference between kubectl apply and kubectl create?

**Answer:**

| Command | Behavior | Use Case |
|---------|----------|----------|
| `kubectl create` | Imperative, creates new resource | One-time creation |
| `kubectl apply` | Declarative, creates or updates | GitOps, declarative management |

**Examples:**
```bash
# Imperative
kubectl create deployment nginx --image=nginx

# Declarative
kubectl apply -f deployment.yaml
```

**Best practice:** Use `kubectl apply` for production as it's declarative and works well with version control.

### 7. What are Labels and Selectors?

**Answer:**
**Labels** are key-value pairs attached to Kubernetes objects for identification and grouping.

**Selectors** are queries used to filter objects based on labels.

**Example:**
```yaml
metadata:
  labels:
    app: frontend
    env: production
    tier: web
```

**Types of selectors:**
- Equality-based: `app=frontend`
- Set-based: `env in (production, staging)`

**Use cases:**
- Service selectors
- Deployment selectors
- Network policies

### 8. What is a ReplicaSet?

**Answer:**
A ReplicaSet ensures a specified number of Pod replicas are running at any given time.

**Key responsibilities:**
- Maintains desired replica count
- Creates new Pods when they fail
- Scales Pods up or down

**Example:**
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
  template:
    # Pod template here
```

**Note:** Usually use Deployments instead of ReplicaSets directly, as Deployments provide update strategies.

### 9. What is the difference between a Deployment and a ReplicaSet?

**Answer:**

| Feature | ReplicaSet | Deployment |
|---------|------------|------------|
| Replica management | Yes | Yes (via ReplicaSet) |
| Rolling updates | No | Yes |
| Rollback capability | No | Yes |
| Update strategies | No | Yes (RollingUpdate, Recreate) |

**Best practice:** Use Deployments for stateless applications.

### 10. What is a Service in Kubernetes?

**Answer:**
A Service is an abstraction that provides a stable network endpoint for accessing Pods. Since Pods are ephemeral, Services provide:
- Stable IP address
- Stable DNS name
- Load balancing across Pods

**Types:**
1. **ClusterIP** (default): Internal cluster access only
2. **NodePort**: Exposes on each node's IP at a static port
3. **LoadBalancer**: Provisions external load balancer (cloud)
4. **ExternalName**: Maps to external DNS name

## Advanced Questions

### 11. What is the role of etcd in Kubernetes?

**Answer:**
etcd is a distributed key-value store that stores all Kubernetes cluster data. It's the "source of truth" for the cluster.

**Key characteristics:**
- Stores cluster state, configuration, and secrets
- Highly available (usually 3, 5, or 7 node quorum)
- Consistent and durable
- Uses Raft consensus algorithm

**Best practices:**
- Regular backups
- Monitor etcd health closely
- Use dedicated disks for etcd
- Keep etcd data encrypted at rest

### 12. How does Kubernetes achieve high availability?

**Answer:**
Kubernetes achieves HA through:

1. **Control Plane HA:**
   - Multiple API servers behind a load balancer
   - Multiple scheduler and controller manager instances (leader election)
   - etcd cluster (quorum-based)

2. **Worker Node HA:**
   - Multiple worker nodes
   - Replication of Pods across nodes
   - Node failure detection and rescheduling

3. **Application HA:**
   - ReplicaSets/Deployments
   - Pod anti-affinity
   - PodDisruptionBudgets

### 13. What is the Kubernetes API Server?

**Answer:**
The API Server is the frontend for the Kubernetes control plane. It:
- Exposes the Kubernetes API
- Handles authentication, authorization, admission control
- Validates and configures data for API objects
- Acts as the gateway to etcd

**Key responsibilities:**
- RESTful API endpoint
- Authentication (certificates, tokens, OIDC)
- Authorization (RBAC, ABAC, Node, Webhook)
- Admission controllers (validation and mutation)

### 14. What happens when you run kubectl get pods?

**Answer:**
1. kubectl reads your kubeconfig file
2. Establishes TLS connection to API server
3. Authenticates using configured credentials
4. Authorization check (RBAC)
5. API server retrieves Pod data from etcd
6. Returns data to kubectl
7. kubectl formats and displays the output

### 15. What are ResourceQuotas and LimitRanges?

**Answer:**
**ResourceQuota:** Limits resource consumption per namespace.
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
spec:
  hard:
    requests.cpu: "10"
    requests.memory: 20Gi
    limits.cpu: "20"
    pods: "10"
```

**LimitRange:** Sets default resource limits for containers.
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-limit-range
spec:
  limits:
  - default:
      cpu: 1
    defaultRequest:
      cpu: 0.5
    type: Container
```

**Use cases:**
- Prevent resource starvation
- Fair resource distribution
- Cost control

## Scenario Questions

### 16. How would you explain Kubernetes to a non-technical person?

**Answer:**
"Imagine you're running a restaurant. Docker containers are like individual chefs, each specialized in making one dish. Kubernetes is the restaurant manager who:
- Decides how many chefs to hire (scaling)
- Assigns chefs to different stations (scheduling)
- If a chef gets sick, calls in a replacement (self-healing)
- Makes sure customers' orders go to the right chefs (load balancing)
- Keeps track of all the recipes (configuration management)
- Ensures health and safety rules are followed (security policies)"

### 17. When would you NOT use Kubernetes?

**Answer:**
Kubernetes might not be suitable when:
- **Simple applications**: Single container apps, static websites
- **Limited resources**: Small teams, development environments
- **Monolithic applications**: Not containerized or microservices
- **Cost-sensitive**: Kubernetes has overhead and complexity cost
- **Small scale**: Few containers, simple deployment needs
- **Limited expertise**: Team lacks Kubernetes knowledge
- **Tight deadline**: Learning curve and setup time

**Alternatives:**
- Docker Compose for simple apps
- PaaS offerings (Heroku, Render, Railway)
- Serverless platforms (AWS Lambda, Cloud Functions)

### 18. What's the difference between Kubernetes and Docker Swarm?

**Answer:**

| Feature | Kubernetes | Docker Swarm |
|---------|------------|--------------|
| Complexity | High | Low |
| Learning curve | Steep | Gentle |
| Scaling | Advanced | Basic |
| Networking | Complex, flexible | Simple |
| Load balancing | Built-in | Built-in |
| Rolling updates | Advanced | Basic |
| Ecosystem | Large | Small |
| Production use | Enterprise-grade | Small-medium |

**Choose Kubernetes for:** Complex applications, large scale, enterprise features
**Choose Docker Swarm for:** Simple deployments, quick setup, smaller teams

### 19. What are init containers?

**Answer:**
Init containers are specialized containers that run before app containers in a Pod. They:
- Run to completion before app containers start
- Run sequentially (one at a time)
- Can have different security settings
- Can use different images

**Use cases:**
- Wait for dependencies (database ready)
- Clone git repositories
- Generate configuration files
- Run setup scripts

**Example:**
```yaml
spec:
  initContainers:
  - name: init-db
    image: busybox
    command: ['sh', '-c', 'until nslookup db; do echo waiting; sleep 2; done']
  containers:
  - name: app
    image: myapp
```

### 20. What is the difference between a sidecar container and an init container?

**Answer:**

| Aspect | Init Container | Sidecar Container |
|--------|----------------|-------------------|
| When runs | Before main container | Alongside main container |
| Lifecycle | Runs to completion | Runs continuously |
| Purpose | Setup/init tasks | Helper/extension tasks |
| Restart policy | Restart if fails | Restart with main container |

**Sidecar use cases:**
- Log collection (Fluentd, Filebeat)
- Proxy (Envoy, Linkerd)
- Config reloader

**Init use cases:**
- Database migration
- Wait for dependencies
- Setup tasks

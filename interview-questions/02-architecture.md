# Kubernetes Architecture - Interview Questions

## Beginner Questions

### 1. What are the main components of the Kubernetes control plane?

**Answer:**
The control plane manages the cluster state:

1. **API Server** - Frontend for the control plane, exposes Kubernetes API
2. **etcd** - Distributed key-value store for cluster data
3. **Scheduler** - Assigns Pods to nodes
4. **Controller Manager** - Runs controller processes
5. **Cloud Controller Manager** - Cloud-specific control logic

**Note:** In some setups, you might also see DNS and optional addons.

### 2. What are the worker node components?

**Answer:**
Worker nodes run the following components:

1. **kubelet** - Agent that communicates with the control plane
2. **kube-proxy** - Network proxy for service abstraction
3. **Container Runtime** - Software to run containers (containerd, CRI-O)

**Additional:**
- kubelet ensures containers run in Pods
- kube-proxy maintains network rules
- Container runtime pulls images and runs containers

### 3. What is the kubelet?

**Answer:**
The kubelet is the primary "node agent" that:
- Watches the API server for Pod assignments
- Ensures containers are running and healthy
- Reports node status to the control plane
- Executes health checks (liveness, readiness probes)
- Mounts volumes and manages config/secrets

**Key responsibilities:**
- Pod lifecycle management
- Node status reporting
- Resource usage monitoring

### 4. What is the kube-scheduler?

**Answer:**
The kube-scheduler assigns Pods to nodes based on:
- Resource requirements (CPU, memory)
- Hardware/software constraints
- Affinity/anti-affinity rules
- Taints and tolerations
- Data locality

**Process:**
1. **Filtering**: Find feasible nodes
2. **Scoring**: Rank feasible nodes
3. **Binding**: Assign Pod to highest-scoring node

### 5. What does the Controller Manager do?

**Answer:**
The Controller Manager runs controller processes:
- **Node Controller**: Monitor node health
- **Replication Controller**: Maintain correct Pod count
- **Endpoints Controller**: Populate Endpoints for Services
- **Service Account Controller**: Create default accounts
- **Token Controller**: Create API access tokens

**Key concept:** Controllers watch the cluster state and make changes to move toward the desired state.

## Intermediate Questions

### 6. Explain the Kubernetes controller pattern

**Answer:**
Controllers implement a "reconciliation loop":
```
while true:
    desired_state = get_desired_state()
    current_state = get_current_state()
    if desired_state != current_state:
        make_changes_to_reconcile()
```

**Example: Deployment Controller**
1. Watches for Deployment changes
2. Creates ReplicaSet with desired replica count
3. Monitors Pod status
4. Scales up/down as needed

**Key principle:** Declarative - specify "what" you want, controller handles "how"

### 7. What is the role of etcd in Kubernetes?

**Answer:**
etcd is the backing store for all cluster data:

**What it stores:**
- Cluster configuration
- Node registry
- Running workloads
- Secrets
- ConfigMaps
- All Kubernetes objects

**Requirements:**
- Strong consistency (uses Raft protocol)
- High availability (3, 5, or 7 member quorum)
- Low latency access
- Regular backups

**Best practices:**
- Dedicated disk for etcd data
- Monitor etcd health
- Regular backups
- Encrypt at rest

### 8. How does the API server handle authentication and authorization?

**Answer:**
**Authentication (who are you?):**
- Client certificates
- Bearer tokens
- OIDC (OpenID Connect)
- Webhook token authentication
- Static tokens/certs (not recommended for production)

**Authorization (what can you do?):**
- RBAC (Role-Based Access Control) - Most common
- ABAC (Attribute-Based Access Control)
- Node authorization
- Webhook mode

**Flow:**
1. User/client sends request with credentials
2. API server authenticates
3. Authorization check (e.g., RBAC)
4. Admission controllers validate/mutate
5. Request processed or rejected

### 9. What are admission controllers?

**Answer:**
Admission controllers intercept requests to the API server before object persistence.

**Types:**
- **Validating**: Check if request is valid
- **Mutating**: Modify the request

**Common examples:**
- `NamespaceExists`: Validate namespace exists
- `ResourceQuota`: Enforce resource limits
- `LimitRanger`: Apply default limits
- `PodSecurity`: Enforce pod security standards
- `ServiceAccount`: Automate service account tokens

**Custom:**
- ValidatingAdmissionWebhook
- MutatingAdmissionWebhook

### 10. How does the scheduler assign Pods to nodes?

**Answer:**
**Phase 1: Filtering**
- Check node resources (CPU, memory)
- Check node selectors and affinity
- Check taints and tolerations
- Check volume binding
- Result: List of feasible nodes

**Phase 2: Scoring**
- Score nodes based on:
  - Resource utilization
  - Affinity rules
  - Anti-affinity rules
  - Custom priorities

**Phase 3: Binding**
- Assign Pod to highest-scoring node
- Update API server with decision

## Advanced Questions

### 11. What is the difference between kube-proxy modes?

**Answer:**

| Mode | How it works | Performance | Features |
|------|--------------|-------------|----------|
| **iptables** | Uses iptables rules | Fast | Most common |
| **IPVS** | Uses IPVS (IP Virtual Server) | Faster for large clusters | More load balancing algorithms |
| **userspace** | Older, proxies in userspace | Slower | Deprecated |

**iptables mode:**
- Creates iptables rules for each Service
- Random load balancing
- Default in most clusters

**IPVS mode:**
- Better performance with many Services
- Supports multiple load balancing algorithms (rr, lc, dh, sh, sed, nq)
- Requires IPVS kernel modules

### 12. How does Kubernetes achieve consensus with etcd?

**Answer:**
etcd uses the **Raft consensus algorithm**:

**Quorum requirements:**
- N nodes can tolerate (N-1)/2 failures
- 3 nodes → tolerate 1 failure
- 5 nodes → tolerate 2 failures
- 7 nodes → tolerate 3 failures

**Leader election:**
1. Nodes start as followers
2. If no heartbeat from leader, become candidate
3. Request votes from other nodes
4. If majority votes, become leader
5. Leader handles all writes

**Consistency:**
- All writes go through leader
- Leader replicates to followers
- Write confirmed after quorum acknowledgment

### 13. What happens when a node fails?

**Answer:**
**Detection:**
1. kubelet stops sending heartbeats
2. Node controller marks node as `NotReady`
3. After `pod-eviction-timeout` (default 5 min), Pods marked for deletion

**Recovery:**
1. Pods scheduled on other nodes
2. ReplicaSet controller creates new Pods
3. Scheduler assigns to healthy nodes

**Note:** StatefulSets and DaemonSets have special handling.

### 14. Explain the concept of leader election in Kubernetes

**Answer:**
Leader election ensures only one instance of a controller runs at a time.

**How it works:**
1. Multiple controller instances start
2. Each tries to create a Lease object
3. First to create becomes leader
4. Leader renews lease periodically
5. If leader fails, others compete

**Use cases:**
- Scheduler (only one active scheduler)
- Controller manager
- Custom controllers

**Implementation:**
```yaml
apiVersion: coordination.k8s.io/v1
kind: Lease
metadata:
  name: kube-scheduler
  namespace: kube-system
spec:
  holderIdentity: scheduler-1
  leaseDurationSeconds: 15
```

### 15. How does Kubernetes handle API versioning?

**Answer:**
Kubernetes API has multiple version levels:

**Version levels:**
- `v1alpha1`, `v1alpha2` - Early development, unstable
- `v1beta1`, `v1beta2` - Pre-release, may change
- `v1` - Stable, backwards compatible

**API groups:**
- Core group: `/api/v1` (pods, services, etc.)
- Named groups: `/apis/apps/v1` (deployments, etc.)

**Version priority:**
- Storage version: What's stored in etcd
- Served versions: What's exposed via API

**Best practices:**
- Use stable versions (`v1`) in production
- Test with beta versions
- Monitor deprecation notices

### 16. What is the static Pod?

**Answer:**
Static Pods are managed by the kubelet directly, not by the API server.

**Characteristics:**
- Defined in manifest files on the node
- Kubelet watches a specific directory
- Cannot be managed via kubectl
- Mirror Pod is created in API server for visibility

**Location:** `/etc/kubernetes/manifests/` (typically)

**Use cases:**
- Control plane components (API server, scheduler, etc.)
- Node-level agents
- Bootstrap clusters

**Example:**
```yaml
# /etc/kubernetes/manifests/kube-apiserver.yaml
apiVersion: v1
kind: Pod
metadata:
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - name: kube-apiserver
    image: k8s.gcr.io/kube-apiserver:v1.28.0
    # ... configuration
```

## Scenario Questions

### 17. How would you design a highly available Kubernetes cluster?

**Answer:**
**Control Plane:**
- 3 or 5 API server instances behind a load balancer
- 3 or 5 etcd nodes (can be co-located with API servers)
- 2+ scheduler instances (leader election)
- 2+ controller manager instances (leader election)

**Worker Nodes:**
- Minimum 3 worker nodes
- Multiple availability zones/regions
- Node auto-scaling

**Application:**
- Pod anti-affinity for critical workloads
- Multiple replicas
- PodDisruptionBudgets

**Networking:**
- CNI with network policies
- Load balancer redundancy

**Storage:**
- Persistent storage with redundancy
- Storage class with replication

### 18. What happens when you delete a Pod?

**Answer:**
1. User sends delete request
2. API server marks Pod as "Terminating"
3. Endpoint controller removes Pod from Service
4. kubelet receives delete notification
5. kubelet sends SIGTERM to containers
6. Containers have `terminationGracePeriodSeconds` (default 30s) to shutdown gracefully
7. After grace period, SIGKILL sent
8. Pod removed from API server

**Best practice:** Implement graceful shutdown in your application.

### 19. Explain how a request flows through Kubernetes

**Answer:**
```
User → kubectl → API Server → Auth → RBAC → Admission Controllers
       ↓
     etcd (store state)
       ↓
   Controllers (watch for changes)
       ↓
   Scheduler (assigns Pods)
       ↓
   kubelet (on worker node)
       ↓
   Container Runtime (runs container)
```

**Detailed flow:**
1. User creates Deployment via kubectl
2. API server validates and stores in etcd
3. Controller Manager sees new Deployment
4. Creates ReplicaSet
5. Scheduler assigns Pods to nodes
6. kubelet on assigned node creates Pod
7. Container runtime pulls image and starts container

### 20. How would you troubleshoot API server issues?

**Answer:**
**Check API server health:**
```bash
kubectl get --raw='/healthz?verbose'
kubectl get componentstatuses
kubectl logs -n kube-system kube-apiserver-<node>
```

**Common issues:**
1. **Certificate issues**
   - Check certificate expiration
   - Verify certificate trust chain

2. **etcd connectivity**
   - Check etcd health
   - Verify network connectivity

3. **Resource exhaustion**
   - Check API server memory/CPU
   - Monitor request rate

4. **Authentication/Authorization**
   - Check kubeconfig
   - Verify RBAC rules

5. **Network issues**
   - Check firewall rules
   - Verify load balancer status

**Tools:**
- API server audit logs
- kubectl with verbose output (`-v=6`)
- Check API server metrics endpoint

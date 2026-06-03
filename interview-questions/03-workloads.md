# Kubernetes Workloads - Interview Questions

## Beginner Questions

### 1. What is the difference between a Pod and a Container?

**Answer:**

| Aspect | Container | Pod |
|--------|-----------|-----|
| Definition | Single runnable unit | Group of containers |
| IP address | Per container | Single IP for all containers |
| Storage | Isolated | Shared volumes |
| Communication | Via network | Via localhost |
| Lifecycle | Single process | Multiple containers |

**Pod characteristics:**
- Smallest deployable unit in K8s
- One or more containers
- Shared network namespace
- Shared storage volumes
- Co-located and co-scheduled

### 2. What is a Deployment?

**Answer:**
A Deployment provides declarative updates for Pods and ReplicaSets.

**Features:**
- Manage ReplicaSets
- Rolling updates
- Rollbacks
- Scaling

**Example:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
```

### 3. What is a StatefulSet?

**Answer:**
StatefulSet is for stateful applications requiring:
- Stable network identity
- Stable persistent storage
- Ordered deployment and scaling

**Characteristics:**
- Pod names: `web-0`, `web-1`, `web-2`
- Stable DNS: `web-0.service.namespace.svc.cluster.local`
- Persistent volume per pod
- Ordered rolling updates

**Use cases:**
- Databases (PostgreSQL, MySQL)
- Message queues (Kafka, RabbitMQ)
- Distributed systems (Cassandra, Elasticsearch)

### 4. What is a DaemonSet?

**Answer:**
A DaemonSet ensures a copy of a Pod runs on all (or some) nodes.

**Use cases:**
- Log collection (Fluentd, Filebeat)
- Monitoring agents (Node Exporter)
- Network plugins (CNI)
- Storage plugins

**Example:**
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
  namespace: kube-system
spec:
  selector:
    matchLabels:
      name: fluentd
  template:
    # Pod template
```

### 5. What is a Job in Kubernetes?

**Answer:**
A Job creates Pods that run to completion.

**Types:**
- **Non-parallel**: Single pod, runs once
- **Parallel with fixed count**: Multiple pods, specific completions
- **Parallel with work queue**: Multiple pods, any completion

**Example:**
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  template:
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl", "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
  backoffLimit: 4
```

## Intermediate Questions

### 6. What are the differences between Deployment, StatefulSet, and DaemonSet?

**Answer:**

| Feature | Deployment | StatefulSet | DaemonSet |
|---------|------------|-------------|-----------|
| Use case | Stateless apps | Stateful apps | Node-level services |
| Pod identity | Random | Stable (web-0, web-1) | One per node |
| Storage | Shared or none | Per-pod PVC | Node-local or shared |
| Scaling | Parallel | Ordered | Automatic (per node) |
| Updates | Rolling update | Ordered update | Rolling update |
| Network | Random DNS | Stable DNS | Per-node |

**When to use:**
- **Deployment**: Web servers, APIs, microservices
- **StatefulSet**: Databases, queues, caches
- **DaemonSet**: Log agents, monitoring, network plugins

### 7. What is the difference between a Job and a CronJob?

**Answer:**

**Job:** Runs once to completion

**CronJob:** Runs Jobs on a schedule

**CronJob example:**
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: backup
spec:
  schedule: "0 2 * * *"  # Daily at 2 AM
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: backup-image
          restartPolicy: OnFailure
```

**CronJob features:**
- Schedule-based execution
- Concurrency policy (Allow, Forbid, Replace)
- History limits
- Starting deadline

### 8. What are Probes in Kubernetes?

**Answer:**
Probes check container health:

**Liveness Probe:** Restart container if failed
```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10
```

**Readiness Probe:** Remove from service if failed
```yaml
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
```

**Startup Probe:** For slow-starting containers
```yaml
startupProbe:
  httpGet:
    path: /healthz
    port: 8080
  failureThreshold: 30
  periodSeconds: 10
```

**Probe types:**
- `httpGet`: HTTP request
- `tcpSocket`: TCP check
- `exec`: Run command
- `grpc`: gRPC health check

### 9. What is the difference between rolling update and recreate strategy?

**Answer:**

**RollingUpdate (default):**
- Gradually update pods
- Max unavailable pods during update
- Zero downtime deployment

```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
```

**Recreate:**
- Delete all old pods, then create new
- Downtime during update
- Simpler, good for single-replica deployments

```yaml
spec:
  strategy:
    type: Recreate
```

**When to use:**
- RollingUpdate: Production, multiple replicas
- Recreate: Dev/test, single replica, incompatible versions

### 10. How do you perform a rollback in Kubernetes?

**Answer:**
```bash
# View rollout history
kubectl rollout history deployment/nginx

# Rollback to previous version
kubectl rollout undo deployment/nginx

# Rollback to specific revision
kubectl rollout undo deployment/nginx --to-revision=2

# Check rollout status
kubectl rollout status deployment/nginx

# Pause rollout
kubectl rollout pause deployment/nginx

# Resume rollout
kubectl rollout resume deployment/nginx
```

**Revision history:**
```bash
# View revision details
kubectl rollout history deployment/nginx --revision=2
```

## Advanced Questions

### 11. What are Pod Disruption Budgets (PDB)?

**Answer:**
PDB limits the number of Pods that can be down during voluntary disruptions.

**Purpose:**
- Maintain application availability
- Protect against too many Pods being evicted
- Work with cluster maintenance operations

**Example:**
```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: nginx-pdb
spec:
  minAvailable: 2  # Or use maxUnavailable
  selector:
    matchLabels:
      app: nginx
```

**When it applies:**
- Node drains
- Cluster upgrades
- Voluntary disruptions

**When it doesn't apply:**
- Node failure
- Pod crashes
- Involuntary disruptions

### 12. How do you configure resource requests and limits?

**Answer:**
```yaml
resources:
  requests:
    memory: "64Mi"
    cpu: "250m"    # 250 millicores = 0.25 cores
  limits:
    memory: "128Mi"
    cpu: "500m"
```

**Requests:**
- Guaranteed resources
- Used for scheduling decisions
- Always reserve for container

**Limits:**
- Maximum resources allowed
- Container can burst up to limit
- Can lead to OOMKilled if exceeded

**QoS Classes:**
- **Guaranteed**: Requests = Limits (both set)
- **Burstable**: Some requests/limits set
- **BestEffort**: No requests/limits

### 13. What are init containers? When would you use them?

**Answer:**
Init containers run before main containers start.

**Characteristics:**
- Run to completion
- Run sequentially
- Can use different images
- Different security context

**Use cases:**
```yaml
# Wait for service
- name: init-db
  image: busybox
  command: ['sh', '-c', 'until nslookup db-service; do echo waiting; sleep 2; done']

# Clone repository
- name: clone-repo
  image: alpine/git
  command: ['git', 'clone', 'https://github.com/repo.git', '/app']

# Run migrations
- name: migrate
  image: myapp-migrate
  command: ['python', 'manage.py', 'migrate']
```

### 14. Explain Pod lifecycle and phases

**Answer:**
Pod phases:
- **Pending**: Pod accepted, not yet running
- **Running**: At least one container running
- **Succeeded**: All containers terminated successfully
- **Failed**: All containers terminated, at least one failed
- **Unknown**: State unknown (communication issue)

**Container states:**
- **Waiting**: Not yet running
- **Running**: Currently executing
- **Terminated**: Execution finished

**Pod conditions:**
```yaml
status:
  conditions:
  - type: Initialized
  - type: Ready
  - type: ContainersReady
  - type: PodScheduled
```

### 15. What are the different restart policies?

**Answer:**

| Policy | Behavior | Use Case |
|--------|----------|----------|
| Always | Always restart | Deployments (default) |
| OnFailure | Restart on failure | Jobs |
| Never | Never restart | Jobs (one-time) |

**Example:**
```yaml
spec:
  restartPolicy: OnFailure  # For Jobs
```

**Note:**
- Deployments/ReplicaSets: Always
- Jobs: OnFailure or Never
- Static Pods: Always or OnFailure

### 16. What are sidecar containers? Give examples

**Answer:**
Sidecar containers run alongside main container in the same Pod.

**Characteristics:**
- Same Pod, different container
- Share network namespace (localhost)
- Share volumes
- Run continuously

**Examples:**

**Log collection:**
```yaml
containers:
- name: app
  image: myapp
  volumeMounts:
  - name: logs
    mountPath: /var/log/app
- name: log-collector
  image: fluentd
  volumeMounts:
  - name: logs
    mountPath: /var/log/app
```

**Proxy/mesh:**
```yaml
containers:
- name: app
  image: myapp
- name: envoy-proxy
  image: envoyproxy/envoy
```

**Config reloader:**
```yaml
containers:
- name: nginx
  image: nginx
- name: config-reloader
  image: config-reloader
```

## Scenario Questions

### 17. How would you deploy a stateless web application?

**Answer:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
      - name: web
        image: nginx:1.21
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 200m
            memory: 256Mi
        livenessProbe:
          httpGet:
            path: /
            port: 80
        readinessProbe:
          httpGet:
            path: /
            port: 80
---
apiVersion: v1
kind: Service
metadata:
  name: web-app
spec:
  selector:
    app: web-app
  ports:
  - port: 80
    targetPort: 80
```

### 18. How would you deploy a PostgreSQL database?

**Answer:**
Use StatefulSet for stable identity and storage:
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  serviceName: postgres-headless
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:14
        ports:
        - containerPort: 5432
        env:
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: postgres-secret
              key: password
        volumeMounts:
        - name: postgres-data
          mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
  - metadata:
      name: postgres-data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 10Gi
---
apiVersion: v1
kind: Service
metadata:
  name: postgres-headless
spec:
  clusterIP: None
  selector:
    app: postgres
  ports:
  - port: 5432
```

### 19. A Pod is stuck in CrashLoopBackOff. How do you debug?

**Answer:**
**Step 1: Check Pod status**
```bash
kubectl describe pod <pod-name>
```

**Step 2: Check logs**
```bash
kubectl logs <pod-name>
kubectl logs <pod-name> --previous  # Previous container
```

**Step 3: Common causes:**
- Application crash (check logs)
- Missing dependencies
- Configuration errors
- Resource constraints (OOMKilled)
- Failed health checks

**Step 4: Debug pod**
```bash
kubectl debug <pod-name> -it --image=busybox
kubectl exec -it <pod-name> -- sh
```

**Step 5: Check events**
```bash
kubectl get events --field-selector involvedObject.name=<pod-name>
```

### 20. How do you update a Deployment without downtime?

**Answer:**
**Strategy: RollingUpdate (default)**

```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1        # Max pods over desired count
      maxUnavailable: 0  # Keep all pods available
```

**Best practices:**
1. **Set readiness probes**
2. **Use Pod Disruption Budgets**
3. **Monitor rollout**
```bash
kubectl rollout status deployment/<name>
```

4. **Test before full rollout**
```bash
kubectl patch deployment <name> -p '{"spec":{"replicas":2}}'
```

5. **Rollback if issues**
```bash
kubectl rollout undo deployment/<name>
```

6. **Use canary deployments** for risk mitigation

**Example canary:**
```bash
# Main deployment (90%)
kubectl apply -f deployment.yaml

# Canary deployment (10%)
kubectl apply -f deployment-canary.yaml
```

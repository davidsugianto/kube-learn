# Workloads

Workloads are objects that manage your applications in Kubernetes.

## Pod

The smallest deployable unit. Rarely created directly - use controllers instead.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.21
    ports:
    - containerPort: 80
```

### Multi-Container Patterns

**Sidecar** - Helper container
```yaml
spec:
  containers:
  - name: app
    image: myapp:latest
  - name: log-collector
    image: log-collector:latest
```

**Ambassador** - Proxy pattern
**Adapter** - Normalize output

### Pod Lifecycle

- **Pending** - Pod accepted, containers not yet running
- **Running** - At least one container running
- **Succeeded** - All containers terminated successfully
- **Failed** - All containers terminated, at least one failed
- **Unknown** - State unknown

### Init Containers

Run before main containers.

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

### Probes

**Liveness** - Restart container if failed
```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10
```

**Readiness** - Remove from service if failed
```yaml
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
```

**Startup** - For slow-starting containers
```yaml
startupProbe:
  httpGet:
    path: /health
    port: 8080
  failureThreshold: 30
  periodSeconds: 10
```

## ReplicaSet

Maintains a stable set of replica Pods. Use Deployment instead.

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
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: app
        image: myapp:v1
```

## Deployment

Manages ReplicaSets and provides declarative updates.

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
        ports:
        - containerPort: 80
```

### Deployment Strategies

**RollingUpdate** (default):
```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1        # Max pods over desired count
      maxUnavailable: 1  # Max unavailable during update
```

**Recreate**:
```yaml
spec:
  strategy:
    type: Recreate  # Delete all, then create new
```

### Rollbacks

```bash
# History
kubectl rollout history deployment/nginx

# Rollback
kubectl rollout undo deployment/nginx

# Rollback to specific revision
kubectl rollout undo deployment/nginx --to-revision=2

# Pause rollout
kubectl rollout pause deployment/nginx

# Resume rollout
kubectl rollout resume deployment/nginx
```

## StatefulSet

For stateful applications with stable identities.

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "web-headless"
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: web
        image: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```

### StatefulSet Features

- Stable network identity: `web-0`, `web-1`, `web-2`
- Stable persistent storage per pod
- Ordered deployment and scaling
- Ordered rolling updates

## DaemonSet

Ensures a copy of a Pod on each (selected) node.

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
    metadata:
      labels:
        name: fluentd
    spec:
      containers:
      - name: fluentd
        image: fluentd:v1.14
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
```

Use cases:
- Log collection (Fluentd, Filebeat)
- Monitoring agents (Prometheus Node Exporter)
- Network plugins (CNI plugins)
- Storage plugins

## Job

Runs to completion.

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
  backoffLimit: 4    # Max retries
  activeDeadlineSeconds: 100  # Timeout
```

### Job Patterns

**Non-parallel** (default): One pod, runs once
**Parallel with fixed count**:
```yaml
spec:
  completions: 3     # Must complete successfully 3 times
  parallelism: 2     # Run 2 pods at a time
```

**Parallel with work queue**:
```yaml
spec:
  completions: 1     # Mark complete when any pod succeeds
  parallelism: 5     # Run up to 5 pods
```

## CronJob

Runs Jobs on a schedule.

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
            command: ["./backup.sh"]
          restartPolicy: OnFailure
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 1
  concurrencyPolicy: Forbid  # Allow, Forbid, Replace
```

## HorizontalPodAutoscaler

Scales workload based on metrics.

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: php-apache
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: php-apache
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```

## Resource Management

### Requests and Limits

```yaml
resources:
  requests:
    memory: "64Mi"
    cpu: "250m"
  limits:
    memory: "128Mi"
    cpu: "500m"
```

**Requests**: Guaranteed resources
**Limits**: Maximum resources allowed

### QoS Classes

- **Guaranteed** - Limits = Requests (both set)
- **Burstable** - Some requests/limits set
- **BestEffort** - No requests/limits

## Comparison Table

| Controller | Use Case | Stable Identity | Ordered |
|------------|----------|-----------------|---------|
| Deployment | Stateless apps | No | No |
| StatefulSet | Stateful apps | Yes | Yes |
| DaemonSet | Per-node agents | No | N/A |
| Job | Run to completion | No | No |
| CronJob | Scheduled jobs | No | No |

## Next Steps

Proceed to [03-networking](../03-networking) to learn about Kubernetes networking.

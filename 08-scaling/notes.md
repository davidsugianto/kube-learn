# Scaling

Strategies and tools for scaling Kubernetes clusters and applications.

## Horizontal Pod Autoscaler (HPA)

Automatically scales pods based on metrics.

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
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 60
```

### Custom Metrics

```yaml
metrics:
- type: Pods
  pods:
    metric:
      name: requests-per-second
    target:
      type: AverageValue
      averageValue: 1k
```

### External Metrics

```yaml
metrics:
- type: External
  external:
    metric:
      name: queue_messages_ready
      selector:
        matchLabels:
          queue: "workerqueue"
    target:
      type: AverageValue
      averageValue: 30
```

### Behavior

```yaml
behavior:
  scaleDown:
    stabilizationWindowSeconds: 300
    policies:
    - type: Percent
      value: 10
      periodSeconds: 60
  scaleUp:
    stabilizationWindowSeconds: 0
    policies:
    - type: Percent
      value: 100
      periodSeconds: 15
    - type: Pods
      value: 4
      periodSeconds: 15
    selectPolicy: Max
```

## Vertical Pod Autoscaler (VPA)

Automatically adjusts pod resource requests.

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: my-app-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  updatePolicy:
    updateMode: "Auto"  # Off, Initial, Recreate, Auto
  resourcePolicy:
    containerPolicies:
    - containerName: "*"
      minAllowed:
        cpu: 100m
        memory: 256Mi
      maxAllowed:
        cpu: 2
        memory: 4Gi
      controlledResources: ["cpu", "memory"]
```

### Update Modes

| Mode | Description |
|------|-------------|
| Off | Only recommendations, no updates |
| Initial | Only on pod creation |
| Recreate | Recreate pods with new resources |
| Auto | Automatically apply recommendations |

## Cluster Autoscaler

Automatically scales cluster nodes.

### AWS

```bash
# Install cluster-autoscaler
kubectl apply -f https://github.com/kubernetes/autoscaler/raw/master/cluster-autoscaler/cloudprovider/aws/examples/cluster-autoscaler.yaml
```

### GCP

```bash
# GKE: Enable node auto-provisioning
gcloud container clusters update <cluster-name> --enable-autoprovisioning
```

### Configuration

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cluster-autoscaler
  namespace: kube-system
spec:
  template:
    spec:
      containers:
      - image: k8s.gcr.io/autoscaling/cluster-autoscaler:v1.25.0
        name: cluster-autoscaler
        command:
        - ./cluster-autoscaler
        - --scale-down-unneeded-time=10m
        - --scale-down-delay-after-add=10m
        - --scale-down-delay-after-delete=10s
        - --balance-similar-node-groups
        - --expander=priority
```

### Node Group Configuration

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: cluster-autoscaler-priority-expander
  namespace: kube-system
data:
  priorities: |-
    10:
      - .*-high-priority.*
    50:
      - .*-medium-priority.*
```

## KEDA (Kubernetes Event-driven Autoscaling)

Scale based on external events.

```yaml
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: kafka-scaledobject
spec:
  scaleTargetRef:
    name: kafka-consumer
  pollingInterval: 15
  cooldownPeriod: 30
  minReplicaCount: 0
  maxReplicaCount: 10
  triggers:
  - type: kafka
    metadata:
      bootstrapServers: kafka.svc:9092
      consumerGroup: my-group
      topic: orders
      lagThreshold: "10"
```

### Supported Scalers

- Kafka
- RabbitMQ
- AWS SQS
- Azure Service Bus
- Prometheus
- Redis
- PostgreSQL
- Cron
- HTTP

## Scaling Strategies

### Predictive Scaling

Use historical data to predict scaling needs.

```yaml
# Using custom metrics
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: predictive-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 2
  maxReplicas: 20
  metrics:
  - type: External
    external:
      metric:
        name: predicted_cpu_usage
      target:
        type: AverageValue
        averageValue: 70
```

### Over-provisioning

Run extra replicas for quick scaling.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: overprovisioning
spec:
  replicas: 2
  template:
    spec:
      containers:
      - name: pause
        image: k8s.gcr.io/pause
        resources:
          requests:
            cpu: "1"
            memory: "1Gi"
      priorityClassName: overprovisioning
---
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: overprovisioning
value: -1
globalDefault: false
description: "Priority class for overprovisioning"
```

### Multi-metric Scaling

```yaml
metrics:
- type: Resource
  resource:
    name: cpu
    target:
      type: Utilization
      averageUtilization: 70
- type: Resource
  resource:
    name: memory
    target:
      type: Utilization
      averageUtilization: 80
- type: Pods
  pods:
    metric:
      name: http_requests
    target:
      type: AverageValue
      averageValue: 1000
```

## Resource Management

### LimitRanges

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

### ResourceQuotas

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
    limits.memory: 40Gi
    pods: "10"
```

## Best Practices

1. **Set resource requests and limits** - Essential for HPA
2. **Use multiple metrics** - CPU, memory, custom
3. **Configure cooldown periods** - Avoid thrashing
4. **Test scaling** - Load test before production
5. **Monitor scaling events** - Track scaling behavior
6. **Use VPA for right-sizing** - Adjust requests automatically
7. **Consider cluster autoscaler** - Scale nodes when needed
8. **Use KEDA for event-driven** - Scale based on queue depth

## Next Steps

Proceed to [09-advanced](../09-advanced) to learn about advanced Kubernetes topics.

# Observability

Monitoring, logging, and debugging Kubernetes clusters and applications.

## Monitoring

### Metrics Server

Collects resource metrics from Kubelets.

```bash
# Install metrics server
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# View metrics
kubectl top nodes
kubectl top pods
kubectl top pods -n <namespace>
```

### Prometheus

Industry-standard monitoring and alerting.

**Architecture:**
- Prometheus Server - Scrapes and stores metrics
- Alertmanager - Handles alerts
- Pushgateway - Accepts short-lived jobs
- Exporters - Expose metrics (node-exporter, kube-state-metrics)

**Installation:**
```bash
# Using Helm
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install prometheus prometheus-community/prometheus
```

**PromQL Examples:**
```promql
# CPU usage rate
rate(container_cpu_usage_seconds_total[5m])

# Memory usage
container_memory_working_set_bytes

# HTTP requests per second
rate(http_requests_total[5m])

# 95th percentile response time
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))
```

### Grafana

Visualization and dashboards.

```bash
# Install Grafana
helm repo add grafana https://grafana.github.io/helm-charts
helm install grafana grafana/grafana
```

**Popular Dashboards:**
- Kubernetes Cluster Monitoring
- Node Exporter Full
- Kubernetes Pods

### Key Metrics to Monitor

**Cluster Level:**
- Node CPU, Memory, Disk usage
- Node network I/O
- Cluster capacity
- Pod count by namespace

**Pod Level:**
- Container CPU usage
- Container memory usage
- Network I/O per pod
- Restart count

**Application Level:**
- Request rate
- Error rate
- Response time (latency)
- Saturation (resource utilization)

## Logging

### Architecture Patterns

**Sidecar Pattern:**
```yaml
spec:
  containers:
  - name: app
    image: myapp
  - name: log-collector
    image: fluent/fluent-bit
    volumeMounts:
    - name: logs
      mountPath: /var/log/app
```

**DaemonSet Pattern:**
- One log collector per node
- Collects logs from all pods
- Examples: Fluentd, Fluent Bit, Filebeat

### EFK Stack (Elasticsearch, Fluentd, Kibana)

```bash
# Install with Helm
helm repo add elastic https://helm.elastic.co
helm install elasticsearch elastic/elasticsearch
helm install kibana elastic/kibana
helm install fluent-bit stable/fluent-bit
```

### Loki Stack

Lightweight log aggregation (Grafana ecosystem).

```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm install loki grafana/loki-stack
```

### Viewing Logs

```bash
# Pod logs
kubectl logs <pod-name>
kubectl logs -f <pod-name>  # follow
kubectl logs --previous <pod-name>  # previous instance
kubectl logs <pod-name> -c <container-name>  # specific container

# Multi-container pod
kubectl logs <pod-name> --all-containers

# Logs from deployment
kubectl logs deployment/<deployment-name>

# Logs with selector
kubectl logs -l app=frontend
```

## Debugging

### kubectl describe

```bash
kubectl describe pod <pod-name>
kubectl describe node <node-name>
kubectl describe deployment <deployment-name>
```

### kubectl get events

```bash
# All events
kubectl get events --sort-by='.lastTimestamp'

# Events for namespace
kubectl get events -n <namespace>

# Events for resource
kubectl get events --field-selector involvedObject.name=<pod-name>

# Watch events
kubectl get events -w
```

### kubectl debug

```yaml
# Debug pod
kubectl debug <pod-name> -it --image=busybox

# Debug node
kubectl debug node/<node-name> -it --image=busybox

# Debug with copy of pod
kubectl debug <pod-name> -it --copy-to=debug-pod --image=busybox
```

### Common Issues

**CrashLoopBackOff:**
```bash
kubectl logs <pod-name> --previous
kubectl describe pod <pod-name>
```

**Pending:**
```bash
kubectl describe pod <pod-name>
# Check: resources, node selectors, taints, PVCs
```

**ImagePullBackOff:**
```bash
kubectl describe pod <pod-name>
# Check: image name, registry credentials, image existence
```

**OOMEvicted:**
```bash
kubectl describe pod <pod-name>
kubectl top pods
# Check: memory limits, memory usage
```

## Tracing

### Jaeger

Distributed tracing platform.

```bash
# Install Jaeger
kubectl create namespace observability
kubectl apply -f https://github.com/jaegertracing/jaeger-operator/releases/download/v1.47.0/jaeger-operator.yaml -n observability
```

### OpenTelemetry

Observability framework for traces, metrics, and logs.

```bash
# Install OpenTelemetry Collector
kubectl apply -f https://github.com/open-telemetry/opentelemetry-operator/releases/latest/download/opentelemetry-operator.yaml
```

## Alerting

### AlertManager Rules

```yaml
groups:
- name: example
  rules:
  - alert: HighMemoryUsage
    expr: container_memory_usage_bytes > 1e9
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: High memory usage detected
```

### Common Alerts

- PodCrashLooping
- PodNotReady
- NodeNotReady
- HighCPUUsage
- HighMemoryUsage
- DiskPressure
- PVCAlmostFull

## Health Checks

### Liveness Probe

Container restart if failed.

```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10
  failureThreshold: 3
```

### Readiness Probe

Remove from service if failed.

```yaml
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
```

## Next Steps

Proceed to [08-scaling](../08-scaling) to learn about scaling strategies.

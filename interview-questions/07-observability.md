# Kubernetes Observability - Interview Questions

## Beginner Questions

### 1. What is observability in Kubernetes?

**Answer:**
Observability is the ability to understand the internal state of a system by examining its outputs.

**Three pillars:**
1. **Metrics**: Numerical measurements (CPU, memory, request rate)
2. **Logs**: Text records of events
3. **Traces**: Request flow across services

### 2. What is the metrics server?

**Answer:**
Metrics Server collects resource metrics from Kubelets and exposes them via the Kubernetes API.

**Use cases:**
- `kubectl top nodes`
- `kubectl top pods`
- Horizontal Pod Autoscaler

### 3. What is Prometheus?

**Answer:**
Prometheus is an open-source monitoring and alerting system.

**Features:**
- Time-series database
- PromQL query language
- Alerting (Alertmanager)
- Service discovery
- Pull-based collection

### 4. What is Grafana?

**Answer:**
Grafana is an open-source visualization and analytics platform.

**Features:**
- Dashboard creation
- Multiple data sources (Prometheus, InfluxDB, etc.)
- Alerting
- Annotations
- Sharing

### 5. What are the different types of logs in Kubernetes?

**Answer:**
1. **Container logs**: Application logs (stdout/stderr)
2. **Pod logs**: kubectl logs
3. **Node logs**: System logs, kubelet logs
4. **Control plane logs**: API server, scheduler, controller manager
5. **Audit logs**: API server requests

## Intermediate Questions

### 6. How do you view Pod logs?

**Answer:**
```bash
# Current logs
kubectl logs <pod-name>

# Follow logs
kubectl logs -f <pod-name>

# Previous container logs
kubectl logs --previous <pod-name>

# Specific container
kubectl logs <pod-name> -c <container-name>

# Multiple containers
kubectl logs <pod-name> --all-containers

# Tail logs
kubectl logs <pod-name> --tail=100

# Since time
kubectl logs <pod-name> --since=1h
```

### 7. What is the EFK stack?

**Answer:**
EFK = Elasticsearch, Fluentd, Kibana

**Components:**
- **Elasticsearch**: Search and analytics engine
- **Fluentd/Fluent Bit**: Log collector and forwarder
- **Kibana**: Visualization and dashboard

**Architecture:**
```
Pods → Fluentd DaemonSet → Elasticsearch → Kibana
```

### 8. What is liveness vs readiness probe?

**Answer:**

| Probe | Purpose | Failure action |
|-------|---------|----------------|
| Liveness | Is the app running? | Restart container |
| Readiness | Is the app ready? | Remove from Service |

**Example:**
```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10

readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
```

### 9. What is distributed tracing?

**Answer:**
Distributed tracing tracks requests as they flow through multiple services.

**Components:**
- **Trace**: End-to-end request journey
- **Span**: Single operation in a trace
- **Context propagation**: Pass trace context between services

**Tools:**
- Jaeger
- Zipkin
- OpenTelemetry

### 10. What is AlertManager?

**Answer:**
AlertManager handles alerts sent by Prometheus.

**Features:**
- Deduplication
- Grouping
- Routing
- Silencing
- Inhibition

**Configuration:**
```yaml
global:
  slack_api_url: 'https://hooks.slack.com/services/xxx'
route:
  receiver: 'slack-notifications'
  routes:
  - match:
      severity: critical
    receiver: 'slack-critical'
receivers:
- name: 'slack-notifications'
  slack_configs:
  - channel: '#alerts'
```

## Advanced Questions

### 11. What is OpenTelemetry?

**Answer:**
OpenTelemetry is a collection of tools, APIs, and SDKs for observability.

**Features:**
- Unified API for traces, metrics, and logs
- Vendor-agnostic
- Multiple language SDKs
- Automatic instrumentation

**Components:**
- API: Define telemetry
- SDK: Implement API
- Collector: Process and export telemetry

### 12. How do you monitor Kubernetes events?

**Answer:**
```bash
# Get events
kubectl get events

# Sort by time
kubectl get events --sort-by='.lastTimestamp'

# Watch events
kubectl get events -w

# Specific namespace
kubectl get events -n <namespace>

# Field selector
kubectl get events --field-selector involvedObject.kind=Pod
```

**Event-based monitoring:**
- Use event exporters
- Send to monitoring system
- Alert on critical events

### 13. What are the key metrics to monitor?

**Answer:**
**Cluster metrics:**
- Node CPU, memory, disk
- Node network I/O
- Cluster capacity
- Pod count

**Pod metrics:**
- Container CPU usage
- Container memory usage
- Network I/O
- Restart count

**Application metrics:**
- Request rate
- Error rate
- Latency/response time
- Saturation (resource utilization)

**Control plane metrics:**
- API server latency
- etcd health
- Scheduler latency
- Controller queue depth

### 14. How do you implement custom metrics?

**Answer:**
**1. Expose metrics endpoint:**
```python
from prometheus_client import Counter, start_http_server

REQUEST_COUNT = Counter('request_count', 'Total requests')

@app.route('/')
def handler():
    REQUEST_COUNT.inc()
    return "Hello"

start_http_server(8080)
```

**2. Deploy Prometheus Adapter:**
```yaml
apiVersion: apiregistration.k8s.io/v1
kind: APIService
metadata:
  name: v1beta1.custom.metrics.k8s.io
spec:
  service:
    name: custom-metrics-apiserver
    namespace: custom-metrics
```

**3. Use in HPA:**
```yaml
metrics:
- type: Pods
  pods:
    metric:
      name: request_count
    target:
      type: AverageValue
      averageValue: 100
```

### 15. What is the USE method?

**Answer:**
USE = Utilization, Saturation, Errors

**For every resource, check:**
- **Utilization**: Percentage of resource used
- **Saturation**: Degree to which resource is over-requested
- **Errors**: Error count

**Example (CPU):**
- Utilization: % CPU time used
- Saturation: Run queue length
- Errors: CPU throttling events

**Example (Memory):**
- Utilization: % memory used
- Saturation: Swap usage
- Errors: OOM kills

### 16. What is the RED method?

**Answer:**
RED = Rate, Errors, Duration

**For every service, check:**
- **Rate**: Requests per second
- **Errors**: Failed requests per second
- **Duration**: Distribution of request latencies

**PromQL examples:**
```promql
# Rate
rate(http_requests_total[5m])

# Errors
rate(http_requests_total{status=~"5.."}[5m])

# Duration (95th percentile)
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))
```

## Scenario Questions

### 17. How do you troubleshoot a slow application?

**Answer:**
**1. Check application metrics:**
- Request rate
- Response time
- Error rate

**2. Check resource metrics:**
```bash
kubectl top pods
kubectl describe pod <pod-name>
```

**3. Check logs:**
```bash
kubectl logs <pod-name> --tail=100
```

**4. Check traces:**
- View distributed traces
- Identify slow operations

**5. Check dependencies:**
- Database queries
- External API calls

**6. Check network:**
- DNS resolution
- Network policies
- Service mesh (if applicable)

### 18. Design a monitoring architecture

**Answer:**
```
┌─────────────────────────────────────────┐
│         Grafana (Visualization)         │
└────────────────┬────────────────────────┘
                 │
        ┌────────┴────────┐
        │                 │
   ┌────▼────┐      ┌────▼─────┐
   │Prometheus│     │ Elasticsearch│
   │(Metrics) │     │  (Logs)    │
   └────┬────┘      └─────┬──────┘
        │                 │
   ┌────▼────┐      ┌────▼─────┐
   │Metrics  │      │  Fluentd │
   │Exporter │      │/Fluent Bit│
   └────┬────┘      └─────┬──────┘
        │                 │
   ┌────▼────────────────▼─────┐
   │   Kubernetes Cluster      │
   │  (Pods, Nodes, Services)  │
   └───────────────────────────┘
```

**Components:**
- Prometheus: Metrics collection
- Grafana: Visualization
- Fluentd: Log collection
- Elasticsearch: Log storage
- Jaeger: Distributed tracing

### 19. How do you set up alerting?

**Answer:**
**1. Define PrometheusRule:**
```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: my-app-alerts
spec:
  groups:
  - name: my-app
    rules:
    - alert: HighErrorRate
      expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.1
      for: 5m
      labels:
        severity: critical
      annotations:
        summary: High error rate detected
        description: Error rate is {{ $value }} req/s
```

**2. Configure AlertManager:**
```yaml
global:
  slack_api_url: 'https://hooks.slack.com/services/xxx'
route:
  receiver: 'slack'
  routes:
  - match:
      severity: critical
    receiver: 'slack-critical'
receivers:
- name: 'slack'
  slack_configs:
  - channel: '#alerts'
```

**3. Test alerts:**
```bash
amtool alert add alertname=TestAlert severity=critical
```

### 20. How do you monitor a multi-cluster environment?

**Answer:**
**Approaches:**

**1. Federated Prometheus:**
- Each cluster has its own Prometheus
- Central Prometheus scrapes from all clusters
- Use federation or Thanos

**2. Thanos:**
- Query across multiple clusters
- Long-term storage
- High availability

**3. Cortex:**
- Horizontally scalable
- Multi-tenant
- Long-term storage

**Architecture (Thanos):**
```
┌──────────────┐  ┌──────────────┐
│  Cluster 1   │  │  Cluster 2   │
│  Prometheus  │  │  Prometheus  │
│   + Thanos   │  │   + Thanos   │
│    Sidecar   │  │    Sidecar   │
└───────┬──────┘  └───────┬──────┘
        │                 │
        └────────┬────────┘
                 │
          ┌──────▼──────┐
          │    Thanos   │
          │    Query    │
          └──────┬──────┘
                 │
          ┌──────▼──────┐
          │   Grafana   │
          └─────────────┘
```

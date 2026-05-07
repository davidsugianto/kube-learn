# Observability - Resources

## Official Documentation

- [Monitoring](https://kubernetes.io/docs/tasks/debug/debug-cluster/resource-usage-monitoring/)
- [Logging Architecture](https://kubernetes.io/docs/concepts/cluster-administration/logging/)
- [Troubleshooting](https://kubernetes.io/docs/tasks/debug/)
- [Application Introspection](https://kubernetes.io/docs/tasks/debug/debug-application/)

## Monitoring Tools

### Prometheus

- [Prometheus](https://prometheus.io/)
- [Prometheus Operator](https://github.com/prometheus-operator/prometheus-operator)
- [Prometheus Helm Chart](https://github.com/prometheus-community/helm-charts/tree/main/charts/prometheus)
- [PromQL Documentation](https://prometheus.io/docs/prometheus/latest/querying/basics/)

### Grafana

- [Grafana](https://grafana.com/)
- [Grafana Helm Chart](https://github.com/grafana/helm-charts/tree/main/charts/grafana)
- [Grafana Dashboards](https://grafana.com/grafana/dashboards/)
- [Kubernetes Dashboard](https://grafana.com/grafana/dashboards/315)

### Metrics Server

- [Metrics Server](https://github.com/kubernetes-sigs/metrics-server)
- [Resource Metrics API](https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/pod-metrics-v1beta1/)

## Logging Tools

### EFK Stack

- [Elasticsearch](https://www.elastic.co/elasticsearch/)
- [Fluentd](https://www.fluentd.org/)
- [Fluent Bit](https://fluentbit.io/)
- [Kibana](https://www.elastic.co/kibana)

### Loki

- [Loki](https://grafana.com/oss/loki/)
- [Loki Helm Chart](https://github.com/grafana/helm-charts/tree/main/charts/loki)
- [Promtail](https://grafana.com/docs/loki/latest/clients/promtail/)

### Other Options

- [Datadog](https://www.datadoghq.com/)
- [Splunk](https://www.splunk.com/)
- [Sumo Logic](https://www.sumologic.com/)
- [Papertrail](https://www.papertrail.com/)

## Tracing

- [Jaeger](https://www.jaegertracing.io/)
- [OpenTelemetry](https://opentelemetry.io/)
- [Zipkin](https://zipkin.io/)
- [Jaeger Operator](https://github.com/jaegertracing/jaeger-operator)

## Debugging Tools

- [kubectl-debug](https://github.com/JamesTGrant/kubectl-debug)
- [ksniff](https://github.com/eldadru/ksniff) - Capture network traffic
- [stern](https://github.com/stern/stern) - Multi-pod log tailing
- [k9s](https://k9scli.io/) - Terminal UI
- [kubectl-tree](https://github.com/ahmetb/kubectl-tree) - Owner references
- [kubectl-graph](https://github.com/steveteuber/kubectl-graph)

## Monitoring Best Practices

- [Monitoring Best Practices](https://kubernetes.io/docs/concepts/cluster-administration/monitoring/)
- [Prometheus Best Practices](https://prometheus.io/docs/practices/)
- [RED Method](https://grafana.com/blog/2018/08/02/the-red-method-how-to-instrument-your-services/)
- [USE Method](http://www.brendangregg.com/usemethod.html)

## Alerting

- [AlertManager](https://prometheus.io/docs/alerting/latest/alertmanager/)
- [Alerting Rules](https://prometheus.io/docs/prometheus/latest/configuration/alerting_rules/)
- [Awesome Prometheus Alerts](https://github.com/samber/awesome-prometheus-alerts)

## Dashboards

- [Kubernetes Cluster Monitoring](https://grafana.com/grafana/dashboards/315)
- [Node Exporter Full](https://grafana.com/grafana/dashboards/1860)
- [Kubernetes Pods](https://grafana.com/grafana/dashboards/6417)
- [Prometheus Operator](https://grafana.com/grafana/dashboards/3662)

## Observability Platforms

- [New Relic](https://newrelic.com/)
- [Dynatrace](https://www.dynatrace.com/)
- [AppDynamics](https://www.appdynamics.com/)
- [Instana](https://www.instana.com/)
- [Honeycomb](https://www.honeycomb.io/)

## Tutorials

- [Troubleshoot Applications](https://kubernetes.io/docs/tasks/debug/debug-application/)
- [Troubleshoot Clusters](https://kubernetes.io/docs/tasks/debug/debug-cluster/)
- [Determine Reason for Pod Failure](https://kubernetes.io/docs/tasks/debug/debug-application/determine-reason-pod-failure/)
- [Debug Services](https://kubernetes.io/docs/tasks/debug/debug-application/debug-service/)

## Books

- **Site Reliability Engineering** - Google SRE Team
- **The Site Reliability Workbook** - Google SRE Team
- **Observability Engineering** - Charity Majors, Liz Fong-Jones, George Miranda

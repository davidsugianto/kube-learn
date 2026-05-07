# Scaling - Resources

## Official Documentation

- [Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)
- [Vertical Pod Autoscaler](https://github.com/kubernetes/autoscaler/tree/master/vertical-pod-autoscaler)
- [Cluster Autoscaler](https://github.com/kubernetes/autoscaler/tree/master/cluster-autoscaler)
- [Resource Management](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)

## HPA Documentation

- [HPA Walkthrough](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/)
- [HPA Algorithm](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/#algorithm-details)
- [HPA API](https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/horizontal-pod-autoscaler-v2/)
- [Metrics Server](https://github.com/kubernetes-sigs/metrics-server)

## VPA Documentation

- [VPA GitHub](https://github.com/kubernetes/autoscaler/tree/master/vertical-pod-autoscaler)
- [VPA Best Practices](https://github.com/kubernetes/autoscaler/blob/master/vertical-pod-autoscaler/docs/best-practices.md)
- [VPA FAQ](https://github.com/kubernetes/autoscaler/blob/master/vertical-pod-autoscaler/docs/faq.md)

## Cluster Autoscaler

- [Cluster Autoscaler GitHub](https://github.com/kubernetes/autoscaler/tree/master/cluster-autoscaler)
- [CA FAQ](https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/FAQ.md)
- [CA Cloud Provider Integrations](https://github.com/kubernetes/autoscaler/tree/master/cluster-autoscaler#deployment)

## KEDA

- [KEDA Documentation](https://keda.sh/docs/)
- [KEDA GitHub](https://github.com/kedacore/keda)
- [KEDA Scalers](https://keda.sh/docs/scalers/)
- [KEDA Helm Chart](https://github.com/kedacore/charts)

## Cloud Provider Autoscaling

### AWS

- [AWS Cluster Autoscaler](https://docs.aws.amazon.com/eks/latest/userguide/cluster-autoscaler.html)
- [AWS Node Groups](https://docs.aws.amazon.com/eks/latest/userguide/managing-node-groups.html)
- [Karpenter](https://karpenter.sh/) - AWS-native autoscaling

### GCP

- [GKE Autoscaling](https://cloud.google.com/kubernetes-engine/docs/concepts/cluster-autoscaler)
- [GKE Node Auto-provisioning](https://cloud.google.com/kubernetes-engine/docs/how-to/node-auto-provisioning)

### Azure

- [AKS Autoscaler](https://docs.microsoft.com/en-us/azure/aks/cluster-autoscaler)
- [AKS Multiple Node Pools](https://docs.microsoft.com/en-us/azure/aks/use-multiple-node-pools)

## Resource Management

- [LimitRanges](https://kubernetes.io/docs/concepts/policy/limit-range/)
- [ResourceQuotas](https://kubernetes.io/docs/concepts/policy/resource-quotas/)
- [Pod Overhead](https://kubernetes.io/docs/concepts/scheduling-eviction/pod-overhead/)

## Monitoring Scaling

- [Monitor HPA](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/#support-for-metrics-apis)
- [Prometheus Adapter](https://github.com/kubernetes-sigs/prometheus-adapter)
- [Custom Metrics](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/#support-for-custom-metrics)

## Best Practices

- [Scaling Best Practices](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/#best-practices)
- [Performance Testing](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/)
- [Resource Quotas Best Practices](https://kubernetes.io/docs/concepts/policy/resource-quotas/#best-practices)

## Tools

- [kube-capacity](https://github.com/robscott/kube-capacity) - Resource capacity CLI
- [kube-resource-report](https://github.com/hjacobs/kube-resource-report) - Resource usage report
- [Goldilocks](https://github.com/FairwindsOps/goldilocks) - VPA recommendations UI

## Tutorials

- [HPA Walkthrough](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/)
- [Scale StatefulSet](https://kubernetes.io/docs/tutorials/stateful-application/scale-statefulset/)
- [Custom Metrics HPA](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/#autoscaling-on-multiple-metrics-and-custom-metrics)

## Books

- **Kubernetes in Action** - Marko Lukša
- **Site Reliability Engineering** - Google SRE Team

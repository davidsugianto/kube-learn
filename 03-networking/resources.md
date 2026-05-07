# Networking - Resources

## Official Documentation

- [Service](https://kubernetes.io/docs/concepts/services-networking/service/)
- [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)
- [Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
- [DNS for Services and Pods](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/)

## Service Documentation

- [Service Types](https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types)
- [Headless Services](https://kubernetes.io/docs/concepts/services-networking/service/#headless-services)
- [Service API](https://kubernetes.io/docs/reference/kubernetes-api/service-resources/service-v1/)
- [EndpointSlices](https://kubernetes.io/docs/concepts/services-networking/endpoint-slices/)

## Ingress Documentation

- [Ingress Controllers](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/)
- [Ingress API](https://kubernetes.io/docs/reference/kubernetes-api/service-resources/ingress-v1/)
- [NGINX Ingress Controller](https://kubernetes.github.io/ingress-nginx/)
- [Multiple Ingress Controllers](https://kubernetes.io/docs/concepts/services-networking/ingress/#multiple-ingress-controllers)

## Network Policy Documentation

- [NetworkPolicy API](https://kubernetes.io/docs/reference/kubernetes-api/policy-resources/network-policy-v1/)
- [Declare Network Policy](https://kubernetes.io/docs/tasks/administer-cluster/declare-network-policy/)
- [Securing Services](https://kubernetes.io/docs/tasks/administer-cluster/securing-services/)

## CNI (Container Network Interface)

- [CNI Plugins](https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/)
- [Calico](https://projectcalico.docs.tigera.io/about/about-calico) - Policy-rich networking
- [Flannel](https://github.com/flannel-io/flannel) - Simple overlay network
- [Weave Net](https://www.weave.works/oss/net/) - Easy to use
- [Cilium](https://cilium.io/) - eBPF-based, high performance
- [Cilium CNI](https://docs.cilium.io/en/stable/)

## Service Mesh

- [Istio](https://istio.io/latest/docs/) - Service mesh
- [Linkerd](https://linkerd.io/) - Lightweight service mesh
- [Consul Connect](https://www.consul.io/docs/connect) - HashiCorp service mesh
- [Kuma](https://kuma.io/docs/) - Universal service mesh

## Ingress Controllers

- [NGINX Ingress Controller](https://kubernetes.github.io/ingress-nginx/)
- [Traefik](https://doc.traefik.io/traefik/)
- [HAProxy Ingress](https://haproxy-ingress.github.io/)
- [AWS ALB Ingress Controller](https://kubernetes-sigs.github.io/aws-load-balancer-controller/)
- [GCP Ingress](https://cloud.google.com/kubernetes-engine/docs/concepts/ingress)

## DNS

- [CoreDNS](https://coredns.io/)
- [CoreDNS Manual](https://coredns.io/manual/toc/)
- [Customizing DNS](https://kubernetes.io/docs/tasks/administer-cluster/dns-custom-nameservers/)

## Load Balancing

- [MetalLB](https://metallb.universe.tf/) - Bare-metal load balancer
- [AWS Load Balancer Controller](https://kubernetes-sigs.github.io/aws-load-balancer-controller/)
- [GCP Load Balancing](https://cloud.google.com/kubernetes-engine/docs/how-to/internal-load-balancing)

## Tutorials

- [Expose Service](https://kubernetes.io/docs/tutorials/stateless-application/expose-external-ip-address/)
- [Connect Frontend to Backend](https://kubernetes.io/docs/tasks/access-application-cluster/connecting-frontend-backend/)
- [Ingress Minikube](https://kubernetes.io/docs/tasks/access-application-cluster/ingress-minikube/)
- [Network Policy Tutorial](https://kubernetes.io/docs/tasks/administer-cluster/declare-network-policy/)

## Books

- **Kubernetes Networking** - James Strong, Vallery Lancey
- **Service Mesh with Istio** - O'Reilly

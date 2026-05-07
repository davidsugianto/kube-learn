# Networking

Kubernetes networking enables communication between pods, services, and external traffic.

## Networking Model

Kubernetes requires:
1. All pods can communicate with all other pods **without NAT**
2. All nodes can communicate with all pods **without NAT**
3. The IP a pod sees itself as is the same IP others see it as

## Service

Abstract way to expose an application running on a set of pods.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
  - port: 80        # Service port
    targetPort: 8080  # Container port
  type: ClusterIP    # Default
```

### Service Types

| Type | Description | Use Case |
|------|-------------|----------|
| ClusterIP | Internal cluster IP only | Internal services |
| NodePort | Exposes on each node's IP | Dev/test, simple external access |
| LoadBalancer | Provisions external LB | Production external access |
| ExternalName | Maps to external DNS | External service alias |

### ClusterIP

Internal service, default type.

```yaml
spec:
  type: ClusterIP
  ports:
  - port: 80
    targetPort: 8080
```

### NodePort

Exposes service on each node's IP at a static port.

```yaml
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 8080
    nodePort: 30007  # Optional: 30000-32767
```

Access: `http://<node-ip>:30007`

### LoadBalancer

Provisions external load balancer (cloud provider).

```yaml
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 8080
```

### Headless Service

No cluster IP, for direct pod access.

```yaml
spec:
  type: ClusterIP
  clusterIP: None  # Makes it headless
```

Use for StatefulSets, direct pod DNS: `pod-name.service-name.namespace.svc.cluster.local`

## Ingress

Manages external access to services, typically HTTP.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: minimal-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: "foo.bar.com"
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: service1
            port:
              number: 80
  - host: "*.foo.bar.com"
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: service2
            port:
              number: 80
```

### Ingress Controllers

Popular options:
- **NGINX Ingress Controller** - Most common
- **Traefik** - Auto-discovery, Let's Encrypt
- **HAProxy** - High performance
- **Istio Gateway** - Service mesh
- **AWS ALB Ingress** - AWS native

## Network Policies

Control traffic flow at the IP address or port level (Layer 3-4).

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-network-policy
  namespace: production
spec:
  podSelector:
    matchLabels:
      role: api
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 8080
  egress:
  - to:
    - podSelector:
        matchLabels:
          role: db
    ports:
    - protocol: TCP
      port: 5432
```

### Default Policies

**Deny all ingress:**
```yaml
spec:
  podSelector: {}
  policyTypes:
  - Ingress
```

**Deny all egress:**
```yaml
spec:
  podSelector: {}
  policyTypes:
  - Egress
```

**Deny all:**
```yaml
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

## DNS

Kubernetes runs internal DNS (CoreDNS).

### Service DNS

- Service: `service-name.namespace.svc.cluster.local`
- Short: `service-name` (same namespace)
- Short: `service-name.namespace`

### Pod DNS

- Pod: `pod-ip-address.namespace.pod.cluster.local`
- Example: `10-0-0-1.default.pod.cluster.local`

### Custom DNS

```yaml
spec:
  dnsPolicy: ClusterFirst  # Default
  dnsConfig:
    nameservers:
    - 8.8.8.8
    searches:
    - my.dns.search
    options:
    - name: ndots
      value: "2"
```

## Service Mesh

Advanced networking with observability and security.

### Popular Options

| Service Mesh | Key Features |
|--------------|--------------|
| **Istio** | Full-featured, most popular |
| **Linkerd** | Lightweight, easy setup |
| **Consul Connect** | HashiCorp ecosystem |
| **Cilium** | eBPF-based, high performance |

### Service Mesh Features

- **Traffic management**: Canary deployments, circuit breaking
- **Security**: mTLS between services
- **Observability**: Distributed tracing, metrics
- **Policy enforcement**: Fine-grained access control

## Comparison Table

| Type | Access | Use Case |
|------|--------|----------|
| ClusterIP | Internal only | Microservices |
| NodePort | Node IP:port | Dev/test |
| LoadBalancer | External LB | Production external |
| Ingress | HTTP/HTTPS routing | Multiple services, host-based routing |
| Service Mesh | Advanced traffic control | Enterprise microservices |

## Next Steps

Proceed to [04-storage](../04-storage) to learn about persistent storage.

# Kubernetes Networking - Interview Questions

## Beginner Questions

### 1. What is the Kubernetes networking model?

**Answer:**
Kubernetes requires:
1. All Pods can communicate with all other Pods without NAT
2. All Nodes can communicate with all Pods without NAT
3. The IP a Pod sees itself as is the same IP others see it as

**Key principles:**
- Flat network namespace
- No address translation between Pods
- Each Pod has its own IP address

### 2. What is a Service in Kubernetes?

**Answer:**
A Service provides stable network access to Pods.

**Types:**
- **ClusterIP** (default): Internal cluster access only
- **NodePort**: Exposes on each node's IP at a static port (30000-32767)
- **LoadBalancer**: Provisions external load balancer (cloud)
- **ExternalName**: Maps to external DNS name

### 3. What is the difference between ClusterIP, NodePort, and LoadBalancer?

**Answer:**

| Type | Access | Use Case |
|------|--------|----------|
| ClusterIP | Internal only | Microservices, databases |
| NodePort | Node IP:port | Dev/test, simple external access |
| LoadBalancer | External LB | Production external access |

**Example:**
```yaml
# ClusterIP (default)
spec:
  type: ClusterIP
  ports:
  - port: 80

# NodePort
spec:
  type: NodePort
  ports:
  - port: 80
    nodePort: 30080

# LoadBalancer
spec:
  type: LoadBalancer
  ports:
  - port: 80
```

### 4. What is kube-proxy?

**Answer:**
kube-proxy maintains network rules on each node for Service abstraction.

**Modes:**
- **iptables**: Uses iptables rules (default)
- **IPVS**: Better performance for many services
- **userspace**: Legacy, not recommended

**Responsibilities:**
- Implement Service abstraction
- Load balance traffic to backend Pods
- Maintain network rules

### 5. What is Ingress?

**Answer:**
Ingress manages external access to Services, typically HTTP/HTTPS.

**Features:**
- URL-based routing
- SSL/TLS termination
- Load balancing
- Name-based virtual hosting

**Example:**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
spec:
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-service
            port:
              number: 80
```

## Intermediate Questions

### 6. What is the difference between Service and Ingress?

**Answer:**

| Feature | Service | Ingress |
|---------|---------|---------|
| Layer | Layer 4 (TCP/UDP) | Layer 7 (HTTP/HTTPS) |
| Routing | By port | By path, host, headers |
| SSL/TLS | No (unless type: LoadBalancer) | Yes (termination) |
| External access | NodePort, LoadBalancer | HTTP/HTTPS only |
| Load balancing | Basic | Advanced (sticky sessions, etc.) |

**When to use:**
- Service: Internal communication, TCP/UDP
- Ingress: HTTP/HTTPS applications, URL routing

### 7. What is a headless Service?

**Answer:**
A headless Service (clusterIP: None) doesn't provide a single IP address.

**Use cases:**
- StatefulSets (stable DNS per Pod)
- Direct Pod-to-Pod communication
- Service discovery without load balancing

**Example:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: headless-service
spec:
  clusterIP: None  # Makes it headless
  selector:
    app: myapp
  ports:
  - port: 80
```

**DNS records:**
- Service: `headless-service.default.svc.cluster.local`
- Pods: `pod-0.headless-service.default.svc.cluster.local`

### 8. What are NetworkPolicies?

**Answer:**
NetworkPolicies control traffic flow at the IP address/port level (Layer 3-4).

**Default behavior:** All traffic allowed (no restrictions)

**Policy types:**
- Ingress: Incoming traffic rules
- Egress: Outgoing traffic rules

**Example:**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-network-policy
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
```

### 9. How does DNS work in Kubernetes?

**Answer:**
Kubernetes runs CoreDNS for internal DNS resolution.

**Service DNS:**
- Full: `<service-name>.<namespace>.svc.cluster.local`
- Short: `<service-name>` (same namespace)
- Namespace: `<service-name>.<namespace>`

**Pod DNS:**
- Format: `<pod-ip>.<namespace>.pod.cluster.local`
- Example: `10-0-0-1.default.pod.cluster.local`

**Custom DNS:**
```yaml
spec:
  dnsPolicy: ClusterFirst
  dnsConfig:
    nameservers:
    - 8.8.8.8
    searches:
    - my.dns.search
```

### 10. What is CNI (Container Network Interface)?

**Answer:**
CNI is a plugin-based networking specification.

**Popular CNI plugins:**
- **Calico**: Policy-rich, BGP routing
- **Flannel**: Simple overlay network
- **Weave Net**: Easy to use
- **Cilium**: eBPF-based, high performance
- **Canal**: Calico + Flannel

**CNI responsibilities:**
- IP address assignment
- Pod network connectivity
- Network policy enforcement (some)

## Advanced Questions

### 11. How does load balancing work in Kubernetes?

**Answer:**
**Service load balancing:**
- kube-proxy maintains rules
- Round-robin by default
- iptables: Random selection
- IPVS: Multiple algorithms (rr, lc, dh, sh, sed, nq)

**Ingress load balancing:**
- Ingress controller handles
- Path-based, host-based routing
- Sticky sessions, health checks

**External load balancing:**
- Cloud provider LB (ELB, GCE LB)
- MetalLB for bare-metal
- NodePort as fallback

### 12. What is the difference between iptables and IPVS mode?

**Answer:**

| Aspect | iptables | IPVS |
|--------|----------|------|
| Performance | Good | Better for many services |
| Load balancing | Random | Multiple algorithms |
| Scalability | Slower with many rules | Faster with many rules |
| Setup | Built-in | Requires kernel modules |

**When to use IPVS:**
- Large number of Services (1000+)
- Need specific LB algorithms
- Performance-critical

**Enable IPVS:**
```bash
kube-proxy --proxy-mode=ipvs
```

### 13. What is a Service Mesh?

**Answer:**
A service mesh provides:
- **Traffic management**: Canary deployments, circuit breaking
- **Security**: mTLS between services
- **Observability**: Distributed tracing, metrics

**Popular options:**
- **Istio**: Full-featured, complex
- **Linkerd**: Lightweight, easy to use
- **Consul Connect**: HashiCorp ecosystem
- **Cilium**: eBPF-based

**Sidecar pattern:**
- Proxy injected into each Pod
- Handles all network traffic
- Example: Envoy proxy

### 14. How does Istio work?

**Answer:**
**Components:**
- **Envoy**: Sidecar proxy
- **Istiod**: Control plane (Pilot, Citadel, Galley)

**Features:**
- Traffic management (routing, splitting)
- Security (mTLS, policies)
- Observability (metrics, tracing)

**Traffic management:**
```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
  - reviews
  http:
  - route:
    - destination:
        host: reviews
        subset: v1
      weight: 90
    - destination:
        host: reviews
        subset: v2
      weight: 10
```

### 15. What is the difference between NodePort and hostPort?

**Answer:**

| Feature | NodePort | hostPort |
|---------|----------|----------|
| Range | 30000-32767 | Any port |
| Binding | On all nodes | On specific node where Pod runs |
| Service | Required | Not required |
| Use case | Service exposure | DaemonSet applications |

**hostPort example:**
```yaml
containers:
- name: app
  ports:
  - containerPort: 80
    hostPort: 8080
```

**When to use:**
- NodePort: Expose Service externally
- hostPort: Bind specific port on node (e.g., monitoring agents)

### 16. How do you secure network traffic in Kubernetes?

**Answer:**
**1. Network Policies:**
```yaml
# Deny all ingress
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
spec:
  podSelector: {}
  policyTypes:
  - Ingress
```

**2. Service Mesh (mTLS):**
- Automatic encryption between services
- Mutual authentication

**3. Ingress TLS:**
```yaml
spec:
  tls:
  - hosts:
    - app.example.com
    secretName: tls-secret
```

**4. Private networking:**
- Use private IP ranges
- Restrict external access

**5. Firewall rules:**
- Control plane access
- Node access

## Scenario Questions

### 17. How would you expose an application externally?

**Answer:**
**Option 1: LoadBalancer Service**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: LoadBalancer
  ports:
  - port: 80
```

**Option 2: Ingress**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
spec:
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /
        backend:
          service:
            name: my-service
            port:
              number: 80
```

**Option 3: NodePort** (dev/test)
```yaml
spec:
  type: NodePort
  ports:
  - port: 80
    nodePort: 30080
```

**Best practice:** Use Ingress for HTTP/HTTPS applications, LoadBalancer for non-HTTP.

### 18. How do you troubleshoot Service connectivity issues?

**Answer:**
**Step 1: Check Service**
```bash
kubectl get svc
kubectl describe svc <service-name>
kubectl get endpoints <service-name>
```

**Step 2: Check Pods**
```bash
kubectl get pods -l app=<label>
kubectl describe pod <pod-name>
```

**Step 3: Test connectivity**
```bash
# From a test pod
kubectl run -it --rm debug --image=busybox -- wget -qO- http://<service-name>:<port>
kubectl run -it --rm debug --image=busybox -- nslookup <service-name>
```

**Step 4: Check DNS**
```bash
kubectl logs -n kube-system -l k8s-app=kube-dns
```

**Step 5: Check network policies**
```bash
kubectl get networkpolicies
```

### 19. Design a microservices networking architecture

**Answer:**
```
                    ┌─────────────┐
                    │   Ingress   │
                    │  Controller │
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │   Service   │
                    │   (API)     │
                    └──────┬──────┘
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
   ┌────▼────┐       ┌────▼────┐       ┌────▼────┐
   │ Service │       │ Service │       │ Service │
   │   (Auth)│       │   (API) │       │  (Data) │
   └────┬────┘       └────┬────┘       └────┬────┘
        │                  │                  │
   ┌────▼────┐       ┌────▼────┐       ┌────▼────┐
   │   Pods  │       │   Pods  │       │   Pods  │
   │  (Auth) │       │   (API) │       │  (Data) │
   └─────────┘       └─────────┘       └─────────┘
```

**Components:**
1. **Ingress**: External traffic routing
2. **Services**: Internal load balancing
3. **NetworkPolicies**: Traffic control
4. **Service Mesh**: mTLS, observability (optional)

### 20. How would you implement canary deployment with networking?

**Answer:**
**Option 1: Service Mesh (Istio)**
```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: my-app
spec:
  hosts:
  - my-app
  http:
  - route:
    - destination:
        host: my-app
        subset: stable
      weight: 90
    - destination:
        host: my-app
        subset: canary
      weight: 10
```

**Option 2: Multiple Services with Ingress**
```yaml
# Stable service
apiVersion: v1
kind: Service
metadata:
  name: my-app-stable
spec:
  selector:
    app: my-app
    version: stable
---
# Canary service
apiVersion: v1
kind: Service
metadata:
  name: my-app-canary
spec:
  selector:
    app: my-app
    version: canary
```

**Option 3: Deployment Replica Ratio**
```bash
# Stable: 9 replicas, Canary: 1 replica
kubectl scale deployment my-app-stable --replicas=9
kubectl scale deployment my-app-canary --replicas=1
```

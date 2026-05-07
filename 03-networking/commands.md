# Networking - Commands

## Service Operations

```bash
# Create service
kubectl expose deployment nginx --port=80 --target-port=80

# Expose as NodePort
kubectl expose deployment nginx --port=80 --target-port=80 --type=NodePort

# Expose as LoadBalancer
kubectl expose deployment nginx --port=80 --target-port=80 --type=LoadBalancer

# Get services
kubectl get services
kubectl get svc  # shorthand

# Describe service
kubectl describe svc <service-name>

# Get service endpoints
kubectl get endpoints <service-name>
kubectl get ep  # shorthand

# Delete service
kubectl delete svc <service-name>
```

## Ingress Operations

```bash
# Get ingress
kubectl get ingress
kubectl get ing  # shorthand

# Describe ingress
kubectl describe ing <ingress-name>

# Get ingress class
kubectl get ingressclass

# Delete ingress
kubectl delete ing <ingress-name>
```

## Network Policy Operations

```bash
# Get network policies
kubectl get networkpolicies
kubectl get netpol  # shorthand

# Describe network policy
kubectl describe netpol <policy-name>

# Delete network policy
kubectl delete netpol <policy-name>
```

## DNS Debugging

```bash
# Test DNS resolution from a pod
kubectl run -it --rm debug --image=busybox -- nslookup kubernetes

# Check DNS from namespace
kubectl run -it --rm debug --image=busybox -- nslookup kubernetes.default

# Test external DNS
kubectl run -it --rm debug --image=busybox -- nslookup google.com

# Check CoreDNS logs
kubectl logs -n kube-system -l k8s-app=kube-dns

# Get CoreDNS pods
kubectl get pods -n kube-system -l k8s-app=kube-dns
```

## Connectivity Testing

```bash
# Test service connectivity
kubectl run -it --rm debug --image=busybox -- wget -qO- http://service-name:80

# Test with curl
kubectl run -it --rm debug --image=curlimages/curl -- curl http://service-name

# Test from specific namespace
kubectl run -it --rm debug -n namespace --image=busybox -- wget -qO- http://service-name.namespace:80

# Test TCP connectivity
kubectl run -it --rm debug --image=busybox -- nc -zv service-name 80
```

## Port Forwarding

```bash
# Forward local port to pod
kubectl port-forward pod/<pod-name> 8080:80

# Forward local port to service
kubectl port-forward svc/<service-name> 8080:80

# Forward to deployment
kubectl port-forward deployment/<deployment-name> 8080:80

# Forward in background
kubectl port-forward svc/<service-name> 8080:80 &

# Forward all addresses
kubectl port-forward --address 0.0.0.0 svc/<service-name> 8080:80
```

## NodePort Access

```bash
# Get node port
kubectl get svc <service-name> -o jsonpath='{.spec.ports[0].nodePort}'

# Get node IP
kubectl get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="InternalIP")].address}'

# Test access
curl http://<node-ip>:<node-port>
```

## Load Balancer

```bash
# Get external IP
kubectl get svc <service-name> -o jsonpath='{.status.loadBalancer.ingress[0].ip}'

# Get hostname (AWS)
kubectl get svc <service-name> -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'

# Watch for IP assignment
kubectl get svc <service-name> -w
```

## Network Debugging

```bash
# Check pod IP
kubectl get pod <pod-name> -o jsonpath='{.status.podIP}'

# Check pod network
kubectl exec <pod-name> -- ip addr
kubectl exec <pod-name> -- route -n

# Check iptables on node (requires node access)
sudo iptables -L -n -v

# Capture network traffic
kubectl exec <pod-name> -- tcpdump -i eth0
```

## Quick YAML Generation

```bash
# Generate ClusterIP service
kubectl expose deployment nginx --port=80 --target-port=80 --dry-run=client -o yaml > service.yaml

# Generate NodePort service
kubectl expose deployment nginx --port=80 --target-port=80 --type=NodePort --dry-run=client -o yaml > service.yaml
```

## Useful Queries

```bash
# All services in cluster
kubectl get svc -A

# Services with external IPs
kubectl get svc -A -o jsonpath='{range .items[?(@.spec.type=="LoadBalancer")]}{.metadata.name}{"\t"}{.status.loadBalancer.ingress[0].ip}{"\n"}{end}'

# All ingress rules
kubectl get ing -A

# Network policies affecting pod
kubectl get netpol -A -o jsonpath='{range .items[?(@.spec.podSelector.matchLabels.app=="nginx")]}{.metadata.name}{"\n"}{end}'
```

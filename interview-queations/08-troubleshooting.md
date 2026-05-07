# Kubernetes Troubleshooting - Interview Questions

## Common Pod Issues

### 1. Pod is stuck in Pending. How do you troubleshoot?

**Answer:**
```bash
# Check events
kubectl describe pod <pod-name>

# Check node resources
kubectl describe node <node-name>
kubectl top nodes

# Check resource quotas
kubectl get resourcequota -n <namespace>

# Common causes:
# 1. Insufficient resources (CPU/memory)
# 2. PersistentVolume not bound
# 3. Node selector/affinity not matching
# 4. Taints and tolerations mismatch
# 5. Pod priority and preemption
```

### 2. Pod is in CrashLoopBackOff. How do you debug?

**Answer:**
```bash
# View logs
kubectl logs <pod-name>
kubectl logs <pod-name> --previous

# Check events
kubectl describe pod <pod-name>

# Debug interactively
kubectl debug <pod-name> -it --image=busybox

# Common causes:
# 1. Application crash
# 2. Missing dependencies
# 3. Configuration errors
# 4. Resource constraints (OOMKilled)
# 5. Failed health checks
```

### 3. Pod is in ImagePullBackOff. What do you check?

**Answer:**
```bash
# Check pod details
kubectl describe pod <pod-name>

# Check image exists
docker pull <image-name>

# Check secrets
kubectl get secrets
kubectl describe pod <pod-name> | grep ImagePullSecrets

# Common causes:
# 1. Wrong image name/tag
# 2. Private registry without credentials
# 3. Image doesn't exist
# 4. Network connectivity issues
# 5. Rate limiting (Docker Hub)
```

### 4. Pod is in OOMKilled state. How do you fix?

**Answer:**
```bash
# Check pod status
kubectl describe pod <pod-name>

# Check resource limits
kubectl get pod <pod-name> -o yaml | grep -A 10 resources

# Check memory usage
kubectl top pods

# Solutions:
# 1. Increase memory limits
# 2. Optimize application memory usage
# 3. Fix memory leaks
# 4. Use appropriate QoS class
```

### 5. Pod is not ready. How do you investigate?

**Answer:**
```bash
# Check readiness probe
kubectl describe pod <pod-name>

# Check container logs
kubectl logs <pod-name>

# Check application health
kubectl exec -it <pod-name> -- curl localhost:8080/healthz

# Common causes:
# 1. Readiness probe failing
# 2. Application not starting
# 3. Dependencies not ready
# 4. Port mismatch
```

## Networking Issues

### 6. Service not working. How do you troubleshoot?

**Answer:**
```bash
# Check service
kubectl get svc
kubectl describe svc <service-name>

# Check endpoints
kubectl get endpoints <service-name>

# Check pod labels
kubectl get pods --show-labels

# Test connectivity
kubectl run -it --rm debug --image=busybox -- wget -qO- http://<service-name>

# Common issues:
# 1. Selector mismatch
# 2. No matching pods
# 3. Wrong port configuration
# 4. Network policies blocking
```

### 7. DNS not resolving. How do you debug?

**Answer:**
```bash
# Test DNS resolution
kubectl run -it --rm debug --image=busybox -- nslookup kubernetes

# Check CoreDNS pods
kubectl get pods -n kube-system -l k8s-app=kube-dns

# Check CoreDNS logs
kubectl logs -n kube-system -l k8s-app=kube-dns

# Check DNS configuration
kubectl exec -it <pod-name> -- cat /etc/resolv.conf

# Common issues:
# 1. CoreDNS not running
# 2. DNS configuration issues
# 3. Network connectivity
# 4. Service not found
```

### 8. Pod cannot connect to external service. How do you debug?

**Answer:**
```bash
# Test connectivity
kubectl exec -it <pod-name> -- ping google.com
kubectl exec -it <pod-name> -- curl -v https://google.com

# Check DNS
kubectl exec -it <pod-name> -- nslookup google.com

# Check network policies
kubectl get networkpolicies -A

# Check iptables
kubectl exec -it <pod-name> -- iptables -L -n -v

# Common issues:
# 1. Egress network policies
# 2. DNS issues
# 3. Firewall rules
# 4. Proxy configuration
```

## Node Issues

### 9. Node is NotReady. How do you troubleshoot?

**Answer:**
```bash
# Check node status
kubectl describe node <node-name>

# Check kubelet status
systemctl status kubelet

# Check kubelet logs
journalctl -u kubelet -f

# Check node resources
kubectl top node <node-name>

# Common issues:
# 1. Kubelet not running
# 2. Out of disk space
# 3. Memory pressure
# 4. Network issues
# 5. Certificate expiration
```

### 10. Node has disk pressure. How do you handle?

**Answer:**
```bash
# Check node condition
kubectl describe node <node-name> | grep -A 5 Conditions

# Check disk usage
df -h

# Clean up resources
docker system prune -a
rm -rf /var/log/pods/*
rm -rf /var/lib/docker/containers/*

# Preventive measures:
# 1. Set up log rotation
# 2. Configure disk quotas
# 3. Monitor disk usage
# 4. Clean up unused images
```

## Performance Issues

### 11. Application is slow. How do you investigate?

**Answer:**
**1. Check metrics:**
```bash
kubectl top pods
kubectl top nodes
```

**2. Check resource limits:**
```bash
kubectl describe pod <pod-name>
```

**3. Check logs:**
```bash
kubectl logs <pod-name> --tail=100
```

**4. Check traces:**
- Use distributed tracing (Jaeger)
- Identify slow operations

**5. Check dependencies:**
- Database queries
- External API calls

**6. Check network:**
- DNS resolution time
- Network latency

### 12. HPA not scaling. How do you debug?

**Answer:**
```bash
# Check HPA status
kubectl describe hpa <hpa-name>

# Check metrics server
kubectl top pods
kubectl get pods -n kube-system -l k8s-app=metrics-server

# Check resource usage
kubectl top pods

# Check HPA events
kubectl get events --field-selector involvedObject.name=<hpa-name>

# Common issues:
# 1. Metrics server not running
# 2. Resource requests not set
# 3. HPA configuration wrong
# 4. Metrics not available
```

## Storage Issues

### 13. PVC is stuck in Pending. How do you troubleshoot?

**Answer:**
```bash
# Check PVC status
kubectl describe pvc <pvc-name>

# Check events
kubectl get events --field-selector involvedObject.name=<pvc-name>

# Check StorageClass
kubectl get storageclass
kubectl describe storageclass <storageclass-name>

# Check PV
kubectl get pv

# Common issues:
# 1. No StorageClass
# 2. Provisioner not running
# 3. Access mode not supported
# 4. Storage quota exceeded
```

### 14. Volume mount fails. How do you debug?

**Answer:**
```bash
# Check pod events
kubectl describe pod <pod-name>

# Check PV and PVC
kubectl get pv,pvc

# Check volume configuration
kubectl get pod <pod-name> -o yaml | grep -A 10 volumes

# Common issues:
# 1. PVC not bound
# 2. Volume path doesn't exist
# 3. Permission issues
# 4. Storage backend issues
```

## Security Issues

### 15. RBAC permission denied. How do you debug?

**Answer:**
```bash
# Check current permissions
kubectl auth can-i --list

# Check specific permission
kubectl auth can-i get pods --as=system:serviceaccount:default:my-sa

# Check role bindings
kubectl get rolebindings,clusterrolebindings -A

# Describe role
kubectl describe role <role-name>
kubectl describe clusterrole <clusterrole-name>

# Common issues:
# 1. Missing Role/RoleBinding
# 2. Wrong namespace
# 3. Incorrect verbs
# 4. Service account not specified
```

### 16. Pod cannot pull from private registry. How do you fix?

**Answer:**
```bash
# Check image pull secrets
kubectl get secrets
kubectl describe pod <pod-name> | grep ImagePullSecrets

# Create image pull secret
kubectl create secret docker-registry regcred \
  --docker-server=<registry> \
  --docker-username=<user> \
  --docker-password=<password>

# Add to pod spec
spec:
  imagePullSecrets:
  - name: regcred

# Verify credentials
kubectl get secret regcred -o jsonpath='{.data.\.dockerconfigjson}' | base64 --decode
```

## Cluster Issues

### 17. API server is slow. How do you troubleshoot?

**Answer:**
```bash
# Check API server health
kubectl get --raw='/healthz?verbose'

# Check API server logs
kubectl logs -n kube-system kube-apiserver-<node-name>

# Check etcd health
etcdctl endpoint health

# Check resource usage
kubectl top nodes

# Common issues:
# 1. High request rate
# 2. etcd performance
# 3. Resource constraints
# 4. Network issues
```

### 18. etcd is slow. How do you debug?

**Answer:**
```bash
# Check etcd health
etcdctl endpoint health
etcdctl endpoint status

# Check etcd metrics
curl -L http://localhost:2379/metrics

# Check disk performance
fio --name=test --filename=/var/lib/etcd/test --sync=1 --rw=write --bs=4k --numjobs=1 --size=1G

# Common issues:
# 1. Disk I/O latency
# 2. Resource constraints
# 3. Large dataset
# 4. Network issues

# Best practices:
# 1. Use SSD
# 2. Dedicated disk
# 3. Defragmentation
# 4. Regular backups
```

## Advanced Debugging

### 19. How do you use kubectl debug?

**Answer:**
```bash
# Debug running pod
kubectl debug <pod-name> -it --image=busybox

# Debug with copy of pod
kubectl debug <pod-name> -it --copy-to=debug-pod --image=busybox

# Debug node
kubectl debug node/<node-name> -it --image=busybox

# Debug with specific container
kubectl debug <pod-name> -it --image=busybox --target=<container-name>
```

### 20. How do you capture network traffic?

**Answer:**
**Method 1: ksniff**
```bash
kubectl plugin install ksniif
kubectl sniff <pod-name> -n <namespace> -o capture.pcap
```

**Method 2: Sidecar container**
```yaml
containers:
- name: app
  image: myapp
- name: tcpdump
  image: corfr/tcpdump
  command: ["tcpdump", "-i", "any", "-w", "/tmp/capture.pcap"]
```

**Method 3: Exec into container**
```bash
kubectl exec -it <pod-name> -- tcpdump -i eth0 -w /tmp/capture.pcap
```

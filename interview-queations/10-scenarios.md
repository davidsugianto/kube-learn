# Scenario-Based Kubernetes Interview Questions

## Architecture & Design

### 1. Design a microservices architecture on Kubernetes

**Requirements:**
- 5 microservices (Auth, API, Web, Data, Cache)
- High availability
- Auto-scaling
- Secure communication
- External access

**Answer:**
```yaml
# Architecture:
# Ingress → Service (Web) → Service (API) → Service (Auth)
#                                       ↓
#                               Service (Data) ↔ Service (Cache)

# Components:
1. Ingress Controller (nginx/traefik)
2. Services (ClusterIP for internal, LoadBalancer for external)
3. Deployments with HPA
4. ConfigMaps and Secrets
5. NetworkPolicies
6. Service Mesh (optional)

# Key considerations:
- Resource limits and requests
- Health checks (liveness/readiness)
- Pod anti-affinity for HA
- Horizontal Pod Autoscaler
- Monitoring and logging
```

### 2. Design a CI/CD pipeline for Kubernetes deployments

**Requirements:**
- Git-based workflow
- Automated testing
- Staged deployments
- Rollback capability

**Answer:**
```yaml
Pipeline Stages:
1. Code commit → Git webhook
2. Build → Container image
3. Test → Unit, integration, e2e
4. Scan → Security, compliance
5. Tag → Version image
6. Deploy to dev → GitOps sync
7. Test in dev → Automated tests
8. Deploy to staging → GitOps sync
9. Manual approval
10. Deploy to production → GitOps sync
11. Monitor → Metrics, logs

Tools:
- Git: GitHub/GitLab
- CI: GitHub Actions/GitLab CI/Jenkins
- CD: ArgoCD/Flux
- Registry: Docker Hub/ECR/GCR
- Testing: pytest, selenium
- Scanning: Trivy, Snyk
```

### 3. How would you migrate a monolithic application to Kubernetes?

**Answer:**
**Phase 1: Assessment**
- Analyze current architecture
- Identify dependencies
- Document configuration
- Check container compatibility

**Phase 2: Containerize**
- Create Dockerfile
- Containerize application
- Set up local testing
- Optimize image size

**Phase 3: Decompose (Strangler Pattern)**
- Identify bounded contexts
- Extract microservices incrementally
- Deploy alongside monolith
- Route traffic gradually

**Phase 4: Migrate Data**
- Plan database migration
- Set up persistent storage
- Implement data sync
- Cutover strategy

**Phase 5: Kubernetes Deployment**
- Create manifests
- Set up services
- Configure ingress
- Implement monitoring

**Phase 6: Cutover**
- Gradual traffic migration
- Monitor performance
- Rollback plan
- Decommission old system

## Performance & Scaling

### 4. Your application is experiencing high latency. How do you diagnose and fix?

**Answer:**
**Diagnosis Steps:**
```bash
1. Check application metrics
   kubectl top pods
   kubectl get pods -o wide

2. Check logs
   kubectl logs <pod-name> --tail=100

3. Check resource usage
   kubectl describe pod <pod-name>

4. Analyze traces
   - Use Jaeger/Zipkin
   - Identify slow operations

5. Check network
   kubectl exec -it <pod-name> -- curl -v http://service
```

**Common Fixes:**
1. **Resource constraints:**
   - Increase CPU/memory limits
   - Add replicas
   - Use HPA

2. **Application issues:**
   - Optimize queries
   - Add caching
   - Fix memory leaks

3. **Network issues:**
   - Check DNS latency
   - Optimize service mesh
   - Add connection pooling

4. **Database issues:**
   - Add indexes
   - Increase connection pool
   - Use read replicas

### 5. Design a solution for handling 100k requests per second

**Answer:**
**Architecture:**
```
DNS → Load Balancer → Ingress → Service → Pods
              ↓
         CDN/Cache
```

**Components:**
1. **Load Balancer:**
   - Cloud LB (ALB/NLB)
   - Global load balancing

2. **Ingress:**
   - Multiple replicas
   - TLS offloading
   - Rate limiting

3. **Application:**
   - StatefulSets for stateful parts
   - Deployments for stateless
   - HPA for auto-scaling
   - Pod anti-affinity

4. **Caching:**
   - Redis cluster
   - CDN for static content

5. **Database:**
   - Read replicas
   - Sharding
   - Connection pooling

6. **Auto-scaling:**
   - HPA based on CPU/custom metrics
   - Cluster autoscaler
   - VPA for right-sizing

7. **Monitoring:**
   - Prometheus + Grafana
   - Distributed tracing
   - Log aggregation

## Security Scenarios

### 6. You discover a vulnerable container image in production. What do you do?

**Answer:**
**Immediate Actions:**
1. Assess severity (CVSS score)
2. Check if vulnerability is exploitable
3. Document findings

**Remediation Steps:**
```bash
1. Find affected pods
   kubectl get pods -A -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.containers[*].image}{"\n"}{end}' | grep <image>

2. Patch the image
   - Update base image
   - Apply security fixes
   - Rebuild and test

3. Deploy updated image
   kubectl set image deployment/<name> <container>=<new-image>

4. Monitor rollout
   kubectl rollout status deployment/<name>

5. Verify fix
   - Rescan image
   - Test functionality
```

**Prevention:**
- Implement image scanning in CI/CD
- Use admission controllers
- Regular vulnerability scans
- Automated image updates

### 7. Implement zero-trust security for a multi-tenant Kubernetes cluster

**Answer:**
**Components:**

1. **Authentication:**
   - OIDC integration
   - Service mesh (mTLS)
   - Certificate management

2. **Authorization:**
   - RBAC per namespace
   - Namespace isolation
   - Resource quotas

3. **Network Security:**
   ```yaml
   # Default deny all
   apiVersion: networking.k8s.io/v1
   kind: NetworkPolicy
   metadata:
     name: default-deny-all
   spec:
     podSelector: {}
     policyTypes:
     - Ingress
     - Egress
   ```

4. **Pod Security:**
   - Pod Security Standards (Restricted)
   - Security contexts
   - Read-only root filesystem

5. **Service Mesh:**
   - mTLS between services
   - Authorization policies
   - Traffic encryption

6. **Audit Logging:**
   - API server audit logs
   - Monitor for anomalies

7. **Secret Management:**
   - External secrets manager
   - Encryption at rest
   - Secret rotation

## Disaster Recovery

### 8. Design a disaster recovery plan for a production cluster

**Answer:**
**RPO/RTO Requirements:**
- RPO (Recovery Point Objective): < 1 hour
- RTO (Recovery Time Objective): < 4 hours

**Backup Strategy:**
```bash
1. etcd backups
   ETCDCTL_API=3 etcdctl snapshot save backup.db

2. Application backups
   - Volume snapshots
   - Database dumps
   - ConfigMaps/Secrets

3. Cluster configuration
   kubectl get all -A -o yaml > cluster-backup.yaml
```

**Recovery Steps:**
1. **Restore etcd:**
   ```bash
   ETCDCTL_API=3 etcdctl snapshot restore backup.db
   ```

2. **Recreate cluster:**
   - Infrastructure as Code
   - Cluster API or similar
   - GitOps for manifests

3. **Restore data:**
   - Restore PVs from snapshots
   - Import database backups

4. **Validate:**
   - Health checks
   - Functionality tests
   - Performance tests

**Multi-Region Setup:**
- Active-passive or active-active
- Cross-region replication
- Global load balancing

### 9. A production database is down. How do you recover?

**Answer:**
**Diagnosis:**
```bash
1. Check pod status
   kubectl get pods -l app=database

2. Check logs
   kubectl logs <db-pod-name> --tail=100

3. Check events
   kubectl describe pod <db-pod-name>

4. Check PVC
   kubectl get pvc
   kubectl describe pvc <pvc-name>
```

**Recovery Options:**

**Option 1: Pod restart**
```bash
kubectl delete pod <db-pod-name>
# StatefulSet will recreate
```

**Option 2: Restore from backup**
```bash
1. Create new PVC from snapshot
   kubectl apply -f pvc-restore.yaml

2. Update StatefulSet to use new PVC

3. Verify data integrity
```

**Option 3: Failover (if replicated)**
```bash
1. Promote replica to primary
2. Update application configuration
3. Verify functionality
```

**Prevention:**
- Regular backups
- Multi-replica setup
- Monitoring and alerting
- Practice recovery procedures

## Troubleshooting Scenarios

### 10. A deployment is failing with "Insufficient cpu". How do you fix it?

**Answer:**
**Diagnosis:**
```bash
# Check node resources
kubectl top nodes
kubectl describe nodes | grep -A 5 "Allocated resources"

# Check resource quotas
kubectl get resourcequota -n <namespace>
kubectl describe resourcequota -n <namespace>
```

**Solutions:**

**Option 1: Reduce resource requests**
```yaml
resources:
  requests:
    cpu: "100m"  # Reduce if over-allocated
    memory: "128Mi"
```

**Option 2: Add nodes**
```bash
# Enable cluster autoscaler
# Or manually add nodes
```

**Option 3: Optimize existing workloads**
```bash
# Find over-provisioned pods
kubectl top pods -A

# Right-size resources
kubectl edit deployment <name>
```

**Option 4: Use LimitRanges**
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: default-limits
spec:
  limits:
  - default:
      cpu: 500m
    defaultRequest:
      cpu: 100m
    type: Container
```

### 11. Services cannot communicate across namespaces. How do you debug?

**Answer:**
**Check connectivity:**
```bash
# Test DNS
kubectl run -it --rm debug --image=busybox -- nslookup <service>.<namespace>

# Test network
kubectl run -it --rm debug --image=curlimages/curl -- curl http://<service>.<namespace>:port
```

**Common issues:**

**1. Network Policies:**
```bash
# Check policies
kubectl get networkpolicies -A

# Allow cross-namespace traffic
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-cross-namespace
spec:
  podSelector: {}
  ingress:
  - from:
    - namespaceSelector: {}
```

**2. Service configuration:**
```yaml
# Use FQDN for cross-namespace
http://<service-name>.<namespace>.svc.cluster.local:port
```

**3. DNS issues:**
```bash
# Check CoreDNS
kubectl logs -n kube-system -l k8s-app=kube-dns
```

## Migration & Upgrades

### 12. Plan a Kubernetes version upgrade with zero downtime

**Answer:**
**Pre-upgrade:**
```bash
1. Backup etcd
2. Check deprecated APIs
   kubectl get apiservices
   kubectl deprecations
3. Review release notes
4. Test in non-production
```

**Upgrade Steps:**

**Control Plane:**
```bash
1. Upgrade first master node
   kubeadm upgrade plan
   kubeadm upgrade apply v1.28.0

2. Verify health
   kubectl get nodes
   kubectl get pods -n kube-system

3. Upgrade other master nodes
```

**Worker Nodes:**
```bash
1. Cordon node
   kubectl cordon <node-name>

2. Drain node
   kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data

3. Upgrade kubelet and kubeadm
   apt-get update
   apt-get install kubelet=<version> kubeadm=<version>
   kubeadm upgrade node

4. Uncordon node
   kubectl uncordon <node-name>

5. Verify node is Ready
   kubectl get nodes
```

**Best Practices:**
- Upgrade one node at a time
- Use PodDisruptionBudgets
- Monitor applications
- Have rollback plan
- Upgrade during low traffic

### 13. Migrate from one cloud provider to another

**Answer:**
**Planning:**
1. Assess current infrastructure
2. Choose migration strategy
3. Plan data migration
4. Estimate costs
5. Create timeline

**Migration Steps:**

**Phase 1: Prepare New Cluster**
```bash
# Provision new cluster
# Set up networking
# Configure storage classes
```

**Phase 2: Deploy Applications**
```bash
# Use GitOps to deploy manifests
# Configure DNS and certificates
# Set up monitoring
```

**Phase 3: Migrate Data**
```bash
# Backup data
# Copy to new infrastructure
# Restore in new cluster
```

**Phase 4: Test**
```bash
# Run tests
# Performance testing
# Security testing
```

**Phase 5: Cutover**
```bash
# Update DNS
# Monitor closely
# Keep old environment ready for rollback
```

**Phase 6: Cleanup**
```bash
# Decommission old cluster
# Clean up resources
```

## Cost Optimization

### 14. How do you reduce Kubernetes costs?

**Answer:**
**Resource Optimization:**
```yaml
1. Right-size pods
   kubectl top pods
   # Adjust requests/limits

2. Use HPA/VPA
   - Scale down when not needed
   - Right-size automatically

3. Implement ResourceQuotas
   apiVersion: v1
   kind: ResourceQuota
   metadata:
     name: compute-quota
   spec:
     hard:
       requests.cpu: "10"
       requests.memory: 20Gi
```

**Infrastructure Optimization:**
```yaml
1. Use spot/preemptible instances
   - For fault-tolerant workloads
   - Save up to 90%

2. Cluster autoscaling
   - Scale down unused nodes

3. Multi-tenant clusters
   - Share resources across teams
```

**Cost Monitoring:**
```yaml
1. Implement tagging
   - By team, project, environment

2. Use cost tools
   - Kubecost
   - Cloud provider tools

3. Regular reviews
   - Identify waste
   - Optimize continuously
```

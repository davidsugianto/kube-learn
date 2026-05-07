# Security - Commands

## RBAC Operations

```bash
# List roles
kubectl get roles -n <namespace>
kubectl get roles -A

# List cluster roles
kubectl get clusterroles

# List role bindings
kubectl get rolebindings -n <namespace>
kubectl get rolebindings -A

# List cluster role bindings
kubectl get clusterrolebindings

# Describe role
kubectl describe role <role-name> -n <namespace>

# Describe cluster role
kubectl describe clusterrole <clusterrole-name>

# Create role from YAML
kubectl apply -f role.yaml

# Create role (imperative)
kubectl create role pod-reader --verb=get,list,watch --resource=pods -n <namespace>

# Create role binding
kubectl create rolebinding read-pods --role=pod-reader --user=jane -n <namespace>

# Create cluster role
kubectl create clusterrole node-reader --verb=get,list,watch --resource=nodes

# Create cluster role binding
kubectl create clusterrolebinding read-nodes --clusterrole=node-reader --user=jane

# Delete role
kubectl delete role <role-name> -n <namespace>

# Delete role binding
kubectl delete rolebinding <rolebinding-name> -n <namespace>
```

## ServiceAccount Operations

```bash
# List service accounts
kubectl get serviceaccounts
kubectl get sa  # shorthand

# Create service account
kubectl create serviceaccount my-sa -n <namespace>

# Describe service account
kubectl describe sa my-sa -n <namespace>

# Get service account token (Kubernetes < 1.24)
kubectl get secret -n <namespace> | grep my-sa-token

# Create token (Kubernetes >= 1.24)
kubectl create token my-sa -n <namespace>

# Create token with expiration
kubectl create token my-sa -n <namespace> --duration=24h

# Delete service account
kubectl delete sa my-sa -n <namespace>
```

## Can-I (Check Permissions)

```bash
# Check if you can perform action
kubectl auth can-i get pods
kubectl auth can-i create deployments

# Check as specific user
kubectl auth can-i get pods --as=jane

# Check as service account
kubectl auth can-i get pods --as=system:serviceaccount:default:my-sa

# Check specific namespace
kubectl auth can-i get pods -n production

# List all allowed actions
kubectl auth can-i --list

# Check resource access
kubectl auth can-i list secrets -n kube-system --as=system:anonymous
```

## Node Security

```bash
# Taint nodes
kubectl taint nodes <node-name> key=value:NoSchedule
kubectl taint nodes <node-name> key=value:NoExecute
kubectl taint nodes <node-name> key=value:PreferNoSchedule

# Remove taint
kubectl taint nodes <node-name> key:NoSchedule-

# List taints
kubectl describe node <node-name> | grep Taints
kubectl get nodes -o custom-columns=NAME:.metadata.name,TAINTS:.spec.taints

# Cordon node (mark unschedulable)
kubectl cordon <node-name>

# Uncordon node (mark schedulable)
kubectl uncordon <node-name>

# Drain node (evict pods)
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data
```

## Pod Security

```bash
# Get pod security context
kubectl get pod <pod-name> -o jsonpath='{.spec.securityContext}'

# Get container security context
kubectl get pod <pod-name> -o jsonpath='{.spec.containers[*].securityContext}'

# Check if pod runs as root
kubectl get pods -A -o jsonpath='{range .items[?(@.spec.securityContext.runAsNonRoot==false)]}{.metadata.name}{"\n"}{end}'

# Check privileged containers
kubectl get pods -A -o jsonpath='{range .items[?(@.spec.containers[*].securityContext.privileged==true)]}{.metadata.name}{"\n"}{end}'
```

## Certificate Management

```bash
# Approve CSR
kubectl get csr
kubectl certificate approve <csr-name>

# Deny CSR
kubectl certificate deny <csr-name>

# Check kubeconfig users
kubectl config view --minify
```

## Image Security

```bash
# List image pull secrets
kubectl get secrets -n <namespace> | grep registry

# Create image pull secret
kubectl create secret docker-registry my-registry \
  --docker-server=registry.example.com \
  --docker-username=user \
  --docker-password=password

# Get pods with image pull secrets
kubectl get pods -A -o jsonpath='{range .items[?(@.spec.imagePullSecrets)]}{.metadata.name}{"\n"}{end}'

# List all images in cluster
kubectl get pods -A -o jsonpath='{range .items[*]}{range .spec.containers[*]}{.image}{"\n"}{end}{end}' | sort | uniq
```

## Audit Logs

```bash
# View kube-apiserver logs
kubectl logs -n kube-system kube-apiserver-<node-name>

# Check audit policy
kubectl get configmap -n kube-system

# View audit logs (requires access to master node)
cat /var/log/kubernetes/audit.log
```

## User Management

```bash
# View current user
kubectl config current-context
kubectl auth whoami

# View contexts
kubectl config get-contexts

# Switch context
kubectl config use-context <context-name>

# Set credentials
kubectl config set-credentials <user-name> --client-certificate=/path/to/cert --client-key=/path/to/key

# Set context
kubectl config set-context <context-name> --cluster=<cluster-name> --user=<user-name> --namespace=<namespace>
```

## Reconcile RBAC

```bash
# Reconcile RBAC from manifest
kubectl auth reconcile -f rbac.yaml

# Check permissions
kubectl auth can-i --list --namespace=<namespace>
```

## Useful Queries

```bash
# All subjects with admin role
kubectl get clusterrolebindings -o jsonpath='{range .items[?(@.roleRef.name=="admin")]}{.metadata.name}{"\t"}{.subjects[*].name}{"\n"}{end}'

# Service accounts with cluster-admin
kubectl get clusterrolebindings -o jsonpath='{range .items[?(@.roleRef.name=="cluster-admin")]}{.subjects[?(@.kind=="ServiceAccount")].name}{"\n"}{end}'

# Pods running as root
kubectl get pods -A -o jsonpath='{range .items[?(@.spec.securityContext.runAsNonRoot!=true)]}{.metadata.name}{"\n"}{end}'

# Pods with privileged containers
kubectl get pods -A -o jsonpath='{range .items[*].spec.containers[?(@.securityContext.privileged==true)]}{.name}{"\n"}{end}'
```

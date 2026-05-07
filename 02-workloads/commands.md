# Workloads - Commands

## Pod Operations

```bash
# Create pod
kubectl run nginx --image=nginx

# Create with resource limits
kubectl run nginx --image=nginx --requests='cpu=100m,memory=128Mi' --limits='cpu=200m,memory=256Mi'

# Create with env vars
kubectl run nginx --image=nginx --env="KEY=VALUE"

# Get pods
kubectl get pods
kubectl get pods -o wide
kubectl get pods --show-labels
kubectl get pods -w  # watch

# Describe pod
kubectl describe pod <pod-name>

# Delete pod
kubectl delete pod <pod-name>
kubectl delete pods --all

# Logs
kubectl logs <pod-name>
kubectl logs -f <pod-name>  # follow
kubectl logs <pod-name> -c <container-name>  # specific container
kubectl logs --previous <pod-name>  # previous instance

# Exec
kubectl exec -it <pod-name> -- sh
kubectl exec <pod-name> -- ls /app
kubectl exec <pod-name> -c <container-name> -- ls /app

# Port forward
kubectl port-forward <pod-name> 8080:80
kubectl port-forward <pod-name> 8080:80 &  # background

# Copy files
kubectl cp <pod-name>:/path/file ./local-file
kubectl cp ./local-file <pod-name>:/path/file
```

## Deployment Operations

```bash
# Create deployment
kubectl create deployment nginx --image=nginx
kubectl create deployment nginx --image=nginx --replicas=3

# Get deployments
kubectl get deployments
kubectl get deploy  # shorthand

# Scale deployment
kubectl scale deployment nginx --replicas=5

# Auto scale
kubectl autoscale deployment nginx --min=2 --max=10 --cpu-percent=80

# Update image
kubectl set image deployment/nginx nginx=nginx:1.22

# View rollout status
kubectl rollout status deployment/nginx

# View rollout history
kubectl rollout history deployment/nginx

# Rollback
kubectl rollout undo deployment/nginx
kubectl rollout undo deployment/nginx --to-revision=2

# Pause/resume rollout
kubectl rollout pause deployment/nginx
kubectl rollout resume deployment/nginx

# Delete deployment
kubectl delete deployment nginx
```

## StatefulSet Operations

```bash
# Get statefulsets
kubectl get statefulsets
kubectl get sts  # shorthand

# Scale statefulset
kubectl scale statefulset web --replicas=5

# Update image
kubectl set image statefulset/web web=nginx:1.22

# Rollout status
kubectl rollout status statefulset/web

# Delete statefulset
kubectl delete statefulset web
```

## DaemonSet Operations

```bash
# Get daemonsets
kubectl get daemonsets
kubectl get ds  # shorthand

# Update image
kubectl set image daemonset/fluentd fluentd=fluentd:v1.15

# Rollout status
kubectl rollout status daemonset/fluentd

# Rollout restart
kubectl rollout restart daemonset/fluentd

# Delete daemonset
kubectl delete daemonset fluentd
```

## Job Operations

```bash
# Create job
kubectl create job pi --image=perl -- perl -Mbignum=bpi -wle 'print bpi(2000)'

# Get jobs
kubectl get jobs

# View job pods
kubectl get pods -l job-name=<job-name>

# Job logs
kubectl logs job/pi

# Delete job
kubectl delete job pi

# Wait for job completion
kubectl wait --for=condition=complete job/pi --timeout=60s
```

## CronJob Operations

```bash
# Create cronjob
kubectl create cronjob backup --image=backup --schedule="0 2 * * *" -- ./backup.sh

# Get cronjobs
kubectl get cronjobs
kubectl get cj  # shorthand

# Manually trigger cronjob
kubectl create job --from=cronjob/backup manual-backup

# Suspend cronjob
kubectl patch cronjob backup -p '{"spec":{"suspend":true}}'

# Delete cronjob
kubectl delete cronjob backup
```

## HPA Operations

```bash
# Create HPA
kubectl autoscale deployment nginx --min=2 --max=10 --cpu-percent=80

# Get HPA
kubectl get hpa

# Describe HPA
kubectl describe hpa nginx

# Edit HPA
kubectl edit hpa nginx

# Delete HPA
kubectl delete hpa nginx
```

## Resource Management

```bash
# View resource usage
kubectl top pods
kubectl top pods -n <namespace>
kubectl top nodes

# View resource quotas
kubectl describe resourcequota -n <namespace>

# View limit ranges
kubectl describe limitrange -n <namespace>
```

## Debugging Workloads

```bash
# Events
kubectl get events --sort-by='.lastTimestamp'
kubectl get events -n <namespace>

# Describe for events
kubectl describe pod <pod-name>

# Check pod issues
kubectl get pods --field-selector=status.phase=Failed
kubectl get pods --field-selector=status.phase=Pending

# Debug with ephemeral container
kubectl debug <pod-name> -it --image=busybox

# Debug node
kubectl debug node/<node-name> -it --image=busybox
```

## Quick YAML Generation

```bash
# Generate pod YAML
kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml

# Generate deployment YAML
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > deployment.yaml

# Generate service YAML
kubectl expose deployment nginx --port=80 --target-port=80 --dry-run=client -o yaml > service.yaml

# Generate job YAML
kubectl create job pi --image=perl --dry-run=client -o yaml -- perl -Mbignum=bpi -wle 'print bpi(2000)' > job.yaml

# Generate cronjob YAML
kubectl create cronjob backup --image=backup --schedule="0 2 * * *" --dry-run=client -o yaml -- ./backup.sh > cronjob.yaml
```

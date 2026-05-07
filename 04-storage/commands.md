# Storage - Commands

## PersistentVolume Operations

```bash
# List PVs
kubectl get pv
kubectl get persistentvolumes

# Get PV details
kubectl describe pv <pv-name>

# Get PV yaml
kubectl get pv <pv-name> -o yaml

# Delete PV
kubectl delete pv <pv-name>
```

## PersistentVolumeClaim Operations

```bash
# List PVCs
kubectl get pvc
kubectl get persistentvolumeclaims

# List PVCs in all namespaces
kubectl get pvc -A

# Get PVC details
kubectl describe pvc <pvc-name>

# Get PVC yaml
kubectl get pvc <pvc-name> -o yaml

# Create PVC
kubectl apply -f pvc.yaml

# Delete PVC
kubectl delete pvc <pvc-name>

# Expand PVC (if supported)
kubectl patch pvc <pvc-name> -p '{"spec":{"resources":{"requests":{"storage":"10Gi"}}}}'
```

## StorageClass Operations

```bash
# List StorageClasses
kubectl get storageclass
kubectl get sc

# Get StorageClass details
kubectl describe sc <storageclass-name>

# Set default StorageClass
kubectl patch storageclass <storageclass-name> -p '{"metadata":{"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'

# Remove default annotation
kubectl patch storageclass <storageclass-name> -p '{"metadata":{"annotations":{"storageclass.kubernetes.io/is-default-class":"false"}}}'

# Delete StorageClass
kubectl delete sc <storageclass-name>
```

## Volume Snapshot Operations

```bash
# List VolumeSnapshotClasses
kubectl get volumesnapshotclass

# List VolumeSnapshots
kubectl get volumesnapshot

# Create snapshot from PVC
kubectl apply -f snapshot.yaml

# Restore from snapshot
kubectl apply -f restore.yaml
```

## Pod Volume Operations

```bash
# Check volumes in pod
kubectl get pod <pod-name> -o jsonpath='{.spec.volumes}'

# Check volume mounts
kubectl get pod <pod-name> -o jsonpath='{.spec.containers[*].volumeMounts}'

# Check if PVC is bound
kubectl get pvc -o jsonpath='{.items[?(@.metadata.name=="my-pvc")].status.phase}'

# Find pods using PVC
kubectl get pods -A -o jsonpath='{range .items[?(@.spec.volumes[*].persistentVolumeClaim.claimName=="my-pvc")]}{.metadata.name}{"\n"}{end}'
```

## Debugging Storage Issues

```bash
# PVC stuck in Pending
kubectl describe pvc <pvc-name>

# Check events
kubectl get events --sort-by='.lastTimestamp'
kubectl get events --field-selector involvedObject.name=<pvc-name>

# Check PV availability
kubectl get pv -o custom-columns=NAME:.metadata.name,CAPACITY:.spec.capacity.storage,STATUS:.status.phase,CLAIM:.spec.claimRef.name

# Check if StorageClass exists
kubectl get sc

# Check provisioner logs
kubectl logs -n kube-system <provisioner-pod-name>

# Test mount in debug pod
kubectl run -it --rm debug --image=busybox -- ls /data
```

## NFS Volumes

```bash
# Create NFS PV
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv
spec:
  capacity:
    storage: 5Gi
  accessModes:
  - ReadWriteMany
  nfs:
    server: 192.168.1.100
    path: /export/data
EOF
```

## HostPath Volumes

```bash
# Create hostPath PV
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolume
metadata:
  name: host-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
  - ReadWriteOnce
  hostPath:
    path: /mnt/data
    type: DirectoryOrCreate
EOF
```

## Quick PVC Creation

```bash
# Create PVC with default StorageClass
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: quick-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
EOF
```

## Useful Queries

```bash
# All PVCs with status
kubectl get pvc -A -o custom-columns=NAMESPACE:.metadata.namespace,NAME:.metadata.name,STATUS:.status.phase,VOLUME:.spec.volumeName

# PVCs by StorageClass
kubectl get pvc -A -o jsonpath='{range .items[?(@.spec.storageClassName=="standard")]}{.metadata.name}{"\n"}{end}'

# Available PVs
kubectl get pv -o jsonpath='{range .items[?(@.status.phase=="Available")]}{.metadata.name}{"\n"}{end}'

# Bound PVs without PVC
kubectl get pv -o jsonpath='{range .items[?(@.status.phase=="Released")]}{.metadata.name}{"\n"}{end}'
```

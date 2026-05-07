# Storage

Persistent storage for stateful applications in Kubernetes.

## Volume Types

### emptyDir

Temporary storage, deleted when pod is removed.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: cache-volume
      mountPath: /cache
  volumes:
  - name: cache-volume
    emptyDir: {}
```

### hostPath

Mounts a file or directory from the host node.

```yaml
volumes:
- name: host-volume
  hostPath:
    path: /var/data
    type: Directory
```

**Types:**
- `Directory` - Existing directory
- `DirectoryOrCreate` - Create if doesn't exist
- `File` - Existing file
- `FileOrCreate` - Create if doesn't exist

### configMap & secret

Mount ConfigMaps and Secrets as volumes.

```yaml
volumes:
- name: config-volume
  configMap:
    name: my-config
- name: secret-volume
  secret:
    secretName: my-secret
```

### PersistentVolumeClaim

Request persistent storage.

```yaml
volumes:
- name: data-volume
  persistentVolumeClaim:
    claimName: my-pvc
```

## PersistentVolume (PV)

Cluster-level storage resource.

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-storage
spec:
  capacity:
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: standard
  hostPath:
    path: /mnt/data
```

### Access Modes

| Mode | Abbreviation | Description |
|------|--------------|-------------|
| ReadWriteOnce | RWO | Single node read-write |
| ReadOnlyMany | ROX | Multiple nodes read-only |
| ReadWriteMany | RWX | Multiple nodes read-write |
| ReadWriteOncePod | RWOP | Single pod read-write |

### Reclaim Policies

| Policy | Behavior |
|--------|----------|
| Retain | Keep data after PVC deletion |
| Delete | Delete storage when PVC is deleted |
| Recycle | Deprecated: scrub and reuse |

### Volume Modes

- `Filesystem` - Mounted as filesystem (default)
- `Block` - Raw block device

## PersistentVolumeClaim (PVC)

Request for storage by a user.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: standard
```

### PVC Binding

1. User creates PVC
2. Controller finds matching PV
3. PVC binds to PV
4. If no PV matches, waits (if dynamic provisioning) or stays pending

## StorageClass

Defines types of storage and provisioners.

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: standard
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-standard
reclaimPolicy: Retain
volumeBindingMode: Immediate
```

### Common Provisioners

| Provisioner | Type |
|-------------|------|
| kubernetes.io/aws-ebs | AWS EBS |
| kubernetes.io/gce-pd | GCP Persistent Disk |
| kubernetes.io/azure-disk | Azure Disk |
| kubernetes.io/cinder | OpenStack Cinder |
| kubernetes.io/rbd | Ceph RBD |
| kubernetes.io/nfs | NFS |
| kubernetes.io/local-path | Local storage |

### Default StorageClass

```yaml
metadata:
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
```

### Volume Binding Modes

| Mode | Description |
|------|-------------|
| Immediate | Bind as soon as created |
| WaitForFirstConsumer | Wait for pod to use it |

## Dynamic Provisioning

Automatically creates PVs when PVCs are created.

```yaml
# PVC triggers automatic PV creation
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dynamic-pvc
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: standard  # Must match StorageClass
  resources:
    requests:
      storage: 5Gi
```

## Using PVCs in Pods

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: data
      mountPath: /data
  volumes:
  - name: data
    persistentVolumeClaim:
      claimName: my-pvc
```

## Volume Expansion

Enable resizing of PVCs.

```yaml
# StorageClass with expansion
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: expandable
provisioner: kubernetes.io/gce-pd
allowVolumeExpansion: true
```

```bash
# Expand PVC
kubectl patch pvc my-pvc -p '{"spec":{"resources":{"requests":{"storage":"10Gi"}}}}'
```

## Volume Snapshots

Requires VolumeSnapshotClass and CSI driver.

```yaml
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshot
metadata:
  name: my-snapshot
spec:
  volumeSnapshotClassName: my-snapshot-class
  source:
    persistentVolumeClaimName: my-pvc
```

## StatefulSet with PVCs

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "web"
  replicas: 3
  template:
    spec:
      containers:
      - name: web
        image: nginx
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```

## Debugging Storage

```bash
# List PVs
kubectl get pv

# List PVCs
kubectl get pvc

# Describe PVC
kubectl describe pvc my-pvc

# List StorageClasses
kubectl get storageclass

# Check events
kubectl get events --field-selector involvedObject.kind=PVC
```

## Next Steps

Proceed to [05-config-secrets](../05-config-secrets) to learn about configuration and secrets management.

# Kubernetes Storage - Interview Questions

## Beginner Questions

### 1. What is a PersistentVolume (PV)?

**Answer:**
A PersistentVolume is a cluster-level storage resource provisioned by an administrator.

**Characteristics:**
- Cluster-scoped resource
- Independent of Pod lifecycle
- Can be static or dynamically provisioned

**Example:**
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-storage
spec:
  capacity:
    storage: 5Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /mnt/data
```

### 2. What is a PersistentVolumeClaim (PVC)?

**Answer:**
A PVC is a request for storage by a user.

**Characteristics:**
- Namespace-scoped
- Requests storage from PV or StorageClass
- Binds to matching PV

**Example:**
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

### 3. What are access modes in Kubernetes storage?

**Answer:**

| Mode | Abbreviation | Description |
|------|--------------|-------------|
| ReadWriteOnce | RWO | Single node read-write |
| ReadOnlyMany | ROX | Multiple nodes read-only |
| ReadWriteMany | RWX | Multiple nodes read-write |
| ReadWriteOncePod | RWOP | Single pod read-write |

**Note:** Not all storage backends support all modes.

### 4. What is a StorageClass?

**Answer:**
A StorageClass defines types of storage and provisions PVs dynamically.

**Example:**
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: standard
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
reclaimPolicy: Delete
```

**Features:**
- Dynamic provisioning
- Different storage tiers
- Reclaim policies
- Volume binding modes

### 5. What is the difference between emptyDir and hostPath?

**Answer:**

| Aspect | emptyDir | hostPath |
|--------|----------|----------|
| Lifecycle | Pod lifecycle | Node lifecycle |
| Storage location | Temporary (RAM/disk) | Node filesystem |
| Data persistence | Lost on Pod deletion | Persists on node |
| Use case | Cache, temp data | Node-level access |

**emptyDir example:**
```yaml
volumes:
- name: cache-volume
  emptyDir: {}
```

**hostPath example:**
```yaml
volumes:
- name: host-volume
  hostPath:
    path: /var/data
```

## Intermediate Questions

### 6. What is dynamic provisioning?

**Answer:**
Dynamic provisioning automatically creates PVs when PVCs are created.

**How it works:**
1. User creates PVC referencing StorageClass
2. Provisioner (in StorageClass) creates PV
3. PV binds to PVC
4. Pod uses the PVC

**Example:**
```yaml
# StorageClass
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-storage
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-ssd
---
# PVC triggers PV creation
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dynamic-pvc
spec:
  storageClassName: fast-storage
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```

### 7. What are reclaim policies?

**Answer:**
Reclaim policies determine what happens to PV after PVC is deleted.

| Policy | Behavior |
|--------|----------|
| Retain | Keep data, requires manual cleanup |
| Delete | Delete storage resource |
| Recycle | Deprecated - scrub and reuse |

**Example:**
```yaml
spec:
  persistentVolumeReclaimPolicy: Retain
```

**Best practices:**
- Retain: Critical data, databases
- Delete: Temporary data, dev/test
- Set default in StorageClass

### 8. What is the difference between static and dynamic provisioning?

**Answer:**

| Aspect | Static Provisioning | Dynamic Provisioning |
|--------|---------------------|----------------------|
| PV creation | Manual (admin) | Automatic |
| Requires StorageClass | No | Yes |
| Flexibility | Limited | High |
| Use case | Specific storage needs | Standard workloads |

**Static provisioning:**
1. Admin creates PV
2. User creates PVC
3. PVC binds to matching PV

**Dynamic provisioning:**
1. User creates PVC with StorageClass
2. PV automatically created
3. PVC binds to PV

### 9. How do you use a PVC in a Pod?

**Answer:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
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

**Key points:**
- Reference PVC in `volumes` section
- Mount volume in `volumeMounts`
- PVC must exist and be bound

### 10. What is a volumeClaimTemplate?

**Answer:**
volumeClaimTemplate is used in StatefulSets to create a PVC per Pod.

**Example:**
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "web-headless"
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

**Result:**
- Creates PVC for each Pod
- PVCs named: `www-web-0`, `www-web-1`, `www-web-2`
- Stable storage even if Pod moves

## Advanced Questions

### 11. What is CSI (Container Storage Interface)?

**Answer:**
CSI is a standard for exposing storage systems to containerized workloads.

**Benefits:**
- Vendor-agnostic interface
- In-tree vs. out-of-tree drivers
- Advanced features (snapshots, resizing)

**Popular CSI drivers:**
- AWS EBS CSI
- GCP PD CSI
- Azure Disk CSI
- Ceph CSI
- Rook-Ceph
- Longhorn
- OpenEBS

**Features:**
- Dynamic provisioning
- Volume snapshots
- Volume cloning
- Volume resizing

### 12. How do you expand a PVC?

**Answer:**
**Step 1: Enable expansion in StorageClass**
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: expandable
provisioner: kubernetes.io/gce-pd
allowVolumeExpansion: true
```

**Step 2: Update PVC**
```bash
kubectl patch pvc my-pvc -p '{"spec":{"resources":{"requests":{"storage":"20Gi"}}}}'
```

**Step 3: Verify**
```bash
kubectl get pvc my-pvc
```

**Note:** Some storage types require Pod restart to see the new size.

### 13. What are VolumeSnapshots?

**Answer:**
VolumeSnapshots allow creating point-in-time copies of volumes.

**Prerequisites:**
- CSI driver with snapshot support
- VolumeSnapshotClass

**Example:**
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

**Restore from snapshot:**
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: restored-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  dataSource:
    name: my-snapshot
    kind: VolumeSnapshot
    apiGroup: snapshot.storage.k8s.io
```

### 14. What is the difference between block and filesystem volumes?

**Answer:**

| Aspect | Filesystem | Block |
|--------|------------|-------|
| Format | Formatted (ext4, xfs) | Raw block device |
| Mount | Mount path | Block device |
| Use case | General purpose | Databases, performance |
| Access | Files | Direct I/O |

**Block volume example:**
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: block-pv
spec:
  capacity:
    storage: 10Gi
  volumeMode: Block
  accessModes:
  - ReadWriteOnce
  fc:
    targetWWNs: ["50060e801049cfd1"]
    lun: 0
```

**Use block for:**
- Databases (MySQL, PostgreSQL)
- High-performance applications
- Raw data storage

### 15. How do you ensure data persistence for stateful applications?

**Answer:**
**1. Use StatefulSets:**
- Stable Pod identities
- Ordered deployment
- Stable storage

**2. Use appropriate StorageClass:**
- High availability (replicated storage)
- Performance requirements
- Backup capabilities

**3. Configure backups:**
- VolumeSnapshots
- Application-level backups
- External backup solutions

**4. Multi-AZ deployment:**
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: multi-az-storage
provisioner: kubernetes.io/aws-ebs
parameters:
  type: io1
  zones: us-east-1a, us-east-1b
```

**5. Use PodDisruptionBudgets:**
```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: db-pdb
spec:
  minAvailable: 1
  selector:
    matchLabels:
      app: database
```

### 16. What are common storage anti-patterns?

**Answer:**
**1. Not setting resource limits:**
- Leads to resource contention
- No QoS guarantees

**2. Using hostPath in production:**
- Node-specific
- Not portable
- Data loss on node failure

**3. Not using PVCs:**
- Direct volume configuration
- Harder to manage
- Not portable

**4. Ignoring backup strategies:**
- No disaster recovery
- Data loss risk

**5. Wrong access mode:**
- RWX for single-node applications
- Performance impact

**6. Not considering storage performance:**
- Wrong storage type
- IOPS limits not considered

## Scenario Questions

### 17. Design storage for a MySQL database

**Answer:**
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: mysql-headless
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: password
        ports:
        - containerPort: 3306
        volumeMounts:
        - name: mysql-data
          mountPath: /var/lib/mysql
        - name: mysql-config
          mountPath: /etc/mysql/conf.d
      volumes:
      - name: mysql-config
        configMap:
          name: mysql-config
  volumeClaimTemplates:
  - metadata:
      name: mysql-data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: fast-ssd
      resources:
        requests:
          storage: 100Gi
```

**Considerations:**
- Use SSD storage for performance
- Configure backups
- Set appropriate IOPS
- Monitor storage usage

### 18. How do you migrate data between PVs?

**Answer:**
**Method 1: Copy using a Pod**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: data-migrator
spec:
  containers:
  - name: migrator
    image: busybox
    command: ['sh', '-c', 'cp -r /source/* /destination/']
    volumeMounts:
    - name: source
      mountPath: /source
    - name: destination
      mountPath: /destination
  volumes:
  - name: source
    persistentVolumeClaim:
      claimName: old-pvc
  - name: destination
    persistentVolumeClaim:
      claimName: new-pvc
```

**Method 2: Use VolumeSnapshots**
```bash
# Create snapshot
kubectl apply -f snapshot.yaml

# Restore to new PVC
kubectl apply -f restore.yaml
```

**Method 3: Application-level migration**
- Backup from old storage
- Restore to new storage
- Update application configuration

### 19. PVC is stuck in Pending. How do you troubleshoot?

**Answer:**
**Step 1: Check PVC status**
```bash
kubectl describe pvc <pvc-name>
```

**Step 2: Check events**
```bash
kubectl get events --field-selector involvedObject.name=<pvc-name>
```

**Step 3: Check StorageClass**
```bash
kubectl get storageclass
```

**Common issues:**
1. **No matching PV** (static provisioning)
   - Create PV with matching size and access mode

2. **StorageClass not found**
   - Check StorageClass exists
   - Verify name in PVC

3. **Provisioner issues**
   - Check provisioner logs
   - Verify cloud provider quota

4. **Access mode mismatch**
   - Check if storage supports requested mode

5. **Insufficient resources**
   - Check cluster storage capacity
   - Check cloud provider limits

### 20. How do you implement storage backup and disaster recovery?

**Answer:**
**1. VolumeSnapshots:**
```yaml
# Schedule regular snapshots
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshot
metadata:
  name: daily-backup
spec:
  volumeSnapshotClassName: daily-snapshot
  source:
    persistentVolumeClaimName: my-pvc
```

**2. Application-level backups:**
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: db-backup
spec:
  schedule: "0 2 * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: backup-image
            command: ["./backup.sh"]
            volumeMounts:
            - name: data
              mountPath: /data
          volumes:
          - name: data
            persistentVolumeClaim:
              claimName: db-pvc
```

**3. External backup solutions:**
- Velero for cluster backups
- Cloud provider backup services
- Third-party backup tools

**4. Disaster recovery:**
- Multi-region replication
- Backup to external storage (S3)
- Regular restore testing
- Documented recovery procedures

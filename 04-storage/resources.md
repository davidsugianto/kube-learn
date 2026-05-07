# Storage - Resources

## Official Documentation

- [Storage Overview](https://kubernetes.io/docs/concepts/storage/)
- [Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
- [Storage Classes](https://kubernetes.io/docs/concepts/storage/storage-classes/)
- [Volume Snapshots](https://kubernetes.io/docs/concepts/storage/volume-snapshots/)

## Volume Types

- [emptyDir](https://kubernetes.io/docs/concepts/storage/volumes/#emptydir)
- [hostPath](https://kubernetes.io/docs/concepts/storage/volumes/#hostpath)
- [local](https://kubernetes.io/docs/concepts/storage/volumes/#local)
- [nfs](https://kubernetes.io/docs/concepts/storage/volumes/#nfs)
- [All Volume Types](https://kubernetes.io/docs/concepts/storage/volumes/)

## Persistent Volumes

- [PV API](https://kubernetes.io/docs/reference/kubernetes-api/config-and-storage-resources/persistent-volume-v1/)
- [PVC API](https://kubernetes.io/docs/reference/kubernetes-api/config-and-storage-resources/persistent-volume-claim-v1/)
- [PVC Protection](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#storage-object-in-use-protection)

## Storage Classes

- [StorageClass API](https://kubernetes.io/docs/reference/kubernetes-api/config-and-storage-resources/storage-class-v1/)
- [Dynamic Provisioning](https://kubernetes.io/docs/concepts/storage/dynamic-provisioning/)
- [Default StorageClass](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#class-1)

## CSI (Container Storage Interface)

- [CSI Overview](https://kubernetes.io/docs/concepts/storage/volumes/#csi)
- [CSI Drivers](https://kubernetes-csi.github.io/docs/drivers.html)
- [CSI Development](https://kubernetes-csi.github.io/docs/)

## Popular CSI Drivers

- [AWS EBS CSI](https://github.com/kubernetes-sigs/aws-ebs-csi-driver)
- [GCP PD CSI](https://github.com/kubernetes-sigs/gcp-compute-persistent-disk-csi-driver)
- [Azure Disk CSI](https://github.com/kubernetes-sigs/azuredisk-csi-driver)
- [Ceph CSI](https://github.com/ceph/ceph-csi)
- [Rook](https://rook.io/) - Storage orchestrator for Kubernetes
- [Longhorn](https://longhorn.io/) - Cloud-native distributed storage
- [OpenEBS](https://openebs.io/) - Container-native storage

## Volume Snapshots

- [VolumeSnapshot API](https://kubernetes.io/docs/reference/kubernetes-api/config-and-storage-resources/volume-snapshot-v1/)
- [VolumeSnapshotClass API](https://kubernetes.io/docs/reference/kubernetes-api/config-and-storage-resources/volume-snapshot-class-v1/)

## Tutorials

- [Configure a Pod to Use a PV for Storage](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/)
- [Dynamic Provisioning](https://kubernetes.io/docs/concepts/storage/dynamic-provisioning/)
- [StatefulSet with Volume](https://kubernetes.io/docs/tutorials/stateful-application/basic-statefulset/)
- [WordPress with Persistent Storage](https://kubernetes.io/docs/tutorials/stateful-application/mysql-wordpress-persistent-volume/)

## Best Practices

- [Storage Best Practices](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#best-practices)
- [Using PVCs](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#claims-as-volumes)
- [Data Protection](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#data-protection)

## Cloud Provider Storage

- [AWS EBS](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-volumes.html)
- [GCP Persistent Disk](https://cloud.google.com/persistent-disk)
- [Azure Disk Storage](https://docs.microsoft.com/en-us/azure/virtual-machines/disks-types)
- [DigitalOcean Block Storage](https://www.digitalocean.com/products/block-storage/)

## Books

- **Storage Patterns in Kubernetes** - Various authors
- **Kubernetes in Action** - Marko Lukša (Chapter on Volumes)

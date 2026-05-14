# [K8S] Config PV and PVC on OpenShift

Owner: Nam Tran
Last edited time: October 14, 2025 3:47 PM

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-sl01y23dms-storage
spec:
  accessModes:
  - ReadWriteMany
  capacity:
    storage: 5Gi
  nfs:
    path: /mnt/Data/K8S/sl01y23dms-storage
    server: fnas.unit.local
  persistentVolumeReclaimPolicy: Retain
  volumeMode: Filesystem
```

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-sl01y23dms-storage
  namespace: sl01y23dms
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 5Gi
  storageClassName: ""
  volumeMode: Filesystem
  volumeName: pv-sl01y23dms-storage
```

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-af01y25phoenix-storage
  namespace: af01y25phoenix
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 10Gi
  storageClassName: nfs-client
  volumeMode: Filesystem
```
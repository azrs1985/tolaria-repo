# [OCP] Config PV and PVC on OpenShift

Owner: Nam Tran
Last edited time: August 28, 2025 1:54 PM

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: <app>-data-pvc
  namespace: <app>
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  volumeName: <app>-data-pv
  storageClassName: ''
  volumeMode: Filesystem
```

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: <app>-data-pv
spec:
  capacity:
    storage: 10Gi
  nfs:
    server: fnas.unit.local
    path: /mnt/Data/<app>-data
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  volumeMode: Filesystem
```
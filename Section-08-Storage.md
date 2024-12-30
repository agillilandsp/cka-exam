# Storage

## Introduction

### Docker Storage

All changes inside a container is preserved outside the image.

Storage Drivers

* AUFS
* ZFS
* BTRFS
* Device Mapper
* Overlay
* Overlay2

## Volume Driver Plugins in Docker

Volumes are not handled in Storage Drivers, but in the Volume Driver Plugins (local is the default).

## Container Storage Interface

### Container Interfaces (cri)

CRI - `rkt` and `cri-o` both are now supported through this interface so there is no longer dependency on Docker.

Similarly the CSI was developed for storage. CNI works the same for Networking

CNI was built for networking similarly.

## Volumes

PV and PVCs are diffferent objects.

PVC is bound to only one PV.

PVCs must match a PV that supports the required Claim:

* Sufficient Capacity
* Access Modes
* Volume Modes
* Storage Class
* Selector

Do not waste space by placing a small PVC on a large PV.

ReclaimPolicy: Delete, Retain, Recycle

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: myfrontend
      image: nginx
      volumeMounts:
      - mountPath: "/var/www/html"
        name: mypd
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: myclaim
```

Lab Notes...

Revisit Question 4 - I made this one too complicated. You can use a hostPath mount from a Pod directly.

I forgot to copy out the yaml files...

ANSWER

```yaml
# Using hostPath
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
  - env:
    - name: LOG_HANDLERS
      value: file
    image: kodecloud/event-simulator
    name: event-simulator
  ...
    volumeMounts:
    - mountPath: /log
      name: log-volume
  ...
  volumes:
  - name: log-volume
    hostPath:
      path: /var/log/webapp
```

```yaml
# pv-log.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-log
spec:
  capacity:
    storage: 100Mi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /pv/log
```

```yaml
# claim-log-1.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: claim-log-1
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 50Mi
```

## Storage Classes

This is the method to filter down PVs that match PVCs

Static Provisioning requires that a volume is allocated before a Pod can run.

Dynamic Provisioning will generate a pv automatically.

PV definition can be retired because the PVCs use of the storage class will qualify the necessary information for the provisioner to generate the PV.

StorageClass... `storageClassName` is qualified in the PVC's manifest.

```yaml
# local-pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: local-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
  storageClassName: local-storage
```

```yaml
# pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  volumes:
  - name: local-pvc
    persistentVolumeClaim:
      claimName: local-pvc
  containers:
  - image: nginx:alpine
    name: nginx
    volumeMounts:
    - mountPath: /var/www/html
      name: local-pvc
```

```yaml
# sc.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: delayed-volume-sc
provisioner: kubernetes.io/no-provisioner
reclaimPolicy: Retain # default value is Delete
volumeBindingMode: WaitForFirstConsumer
```

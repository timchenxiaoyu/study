## kubernetes挂载NFS

PV
```go
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv 
spec:
  capacity:
    storage: 1Gi 
  accessModes:
    - ReadWriteMany 
  persistentVolumeReclaimPolicy: Retain 
  nfs: 
    path: /opt/nfs 
    server: nfs.f22 
    readOnly: false
```

PVC
A persistent volume claim (PVC) specifies the desired access mode and storage capacity. 
Currently, based on only these two attributes, a PVC is bound to a single PV. 

```go
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nfs-pvc  
spec:
  accessModes:
  - ReadWriteMany      
  resources:
     requests:
       storage: 1Gi    
```
但如果PVC没有指定storageClassName，kubernetes将会给他指定一个默认的storageClassName，这么可能会产生冲突
解决的方案是制定空的storageClassName
```go

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nfs
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: ""
  resources:
    requests:
      storage: 1Mi
```
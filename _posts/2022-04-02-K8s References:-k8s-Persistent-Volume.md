
    Created by Ranjith Vijayan on Apr 04, 2022


This post is created for my own reference of k8s resources as part of learning

A PersistentVolume (PV) is a piece of storage in the cluster that has been provisioned by an administrator or dynamically provisioned using Storage Classes.

Kubernetes provides 2 API resources namely PersistentVolume and PersistentVolumeClaim to achieve this

PersistentVolume types are implemented as plugins. Kubernetes currently supports various plugins including cloud service providers specific (AWS EBS,Azure Disk etc), NFS, HostPath etc.


In this chapter we will document a simple HostPath type of Persistent volume. HostPath Volumes are not recommended for production. This is only for Testing. This works only when the same host-path is available on every worker nodes of K8s.

In this example we assume that you already have NFS mount with same path (/scratch/share/kbz-file-share) available on every worker node (so that regardless of which node the pod will be scheduled the file can be shared using NFS mount point) To setup NFS and mount them to the required VMs refer to NFS Server documentation.


#### Create PV

pv.yaml
``` yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-test
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/scratch/share/kbz-file-share"
```

### Apply
``` bash
        kubectl apply -f pv.yaml
```

### Check - kubectl get pv

``` bash
        ¿ kubectl get pv
        NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM                   STORAGECLASS   REASON   AGE
        pv-test                                    10Gi       RWO            Retain           Bound       pv-test/task-pv-claim   manual                  114m
```
### Create Persistence Volume Claim (PVC)
        pvc.yaml
``` yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: task-pv-claim
  namespace: pv-test
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
```

### Check
``` bash
        ¿ kubectl get pvc -n pv-test
        NAME            STATUS   VOLUME    CAPACITY   ACCESS MODES   STORAGECLASS   AGE
        task-pv-claim   Bound    pv-test   10Gi       RWO            manual         115m
```

Example Podcheck

pod.yaml
``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: task-pv-pod
  namespace: pv-test
spec:
  volumes:
    - name: task-pv-storage
      persistentVolumeClaim:
        claimName: task-pv-claim
  containers:
    - name: task-pv-container
      env:
      - name: OB_POD_NAME
        valueFrom:
          fieldRef:
            apiVersion: v1
            fieldPath: metadata.name
      - name: OB_NAMESPACE_NAME
        valueFrom:
          fieldRef:
            apiVersion: v1
            fieldPath: metadata.namespace
      - name: OB_ENV_NAME
        value: sit
      image: fsgbu-mum-128.snbomprshared1.gbucdsint02bom.oraclevcn.com:5000/utils/nginx:latest
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/u01/oracle/logs"
          name: task-pv-storage
          subPathExpr: logs/$(OB_ENV_NAME)/$(OB_NAMESPACE_NAME)/$(OB_POD_NAME)
```
 Note: a subPathExpr is used in this example to mount a sub-directory within the main volume directory. the sub-path will be automatically created if its not present.

### Apply

``` bash
kubectl apply -f pod.yaml
pod/task-pv-pod created
```
### Check
```
kubectl get po -n pv-test   
NAME          READY   STATUS    RESTARTS   AGE
task-pv-pod   1/1     Running   0          86s
```
### Test the directory is mounted and we are able to write file

```
kubectl exec -it -n pv-test task-pv-pod -- sh
# cd /u01/oracle/logs
# ls
# touch abcd.txt
# exit
```         
### check the file is available on the physical host

```         
        ls /scratch/share/kbz-file-share/logs/sit/pv-test/task-pv-pod/        
        abcd.txt
```



    Created by Ranjith Vijayan, last modified on Aug 03, 2022
    
    
In general K8s expects you to design workload with "Cattle" approach, as against the "Pet" approach in the on-prem world. i.e. pods are inter-changeable.

In the "Cattle" approach, your pods can be terminated / replaced / migrated to another node, in order to ensure High-Availability, Self-healing, etc.

However not all applications, especially traditionally designed ones may not be able to handle the abrupt termination of running pods. So its "safer" to allow some grace time to allow inflight transactions to complete, before the pod is removed.


### The Kubernetes termination lifecycle

Once Kubernetes has decided to terminate your pod, a series of events takes place
1. Set to the “Terminating” State

   At this point, the pod stops getting new traffic. Containers running in the pod will not be affected.
2. preStop Hook is executed

   The preStop Hook is a special command or http request that is sent to the containers in the pod. This is useful for the applications that are not designed to handle SIGTERM.
3. SIGTERM signal is sent to the pod

   SIGTERM is the termination signal that is sent to the POD. The application can handle and prepare for graceful shutdown
4. waits for a grace period

   By default k8s gives 30 seconds for your application to honor the SIGTERM. You can configure this using terminationGracePeriod for a different value.
5. SIGKILL signal is sent to pod, and the pod is removed

   SIGKILL is the final signal sent to the POD before its removed. K8s objects are removed


### EXAMPLE:

In the below example, we will see how to use preStop hook. This is generally useful, and is a best practice to delay the termination, thus giving some breather time for the inflight transactions, if any, to complete. In the below example we are going to simply wait for 20 seconds using exec with command: [ "/bin/sleep", "20" ] . (consider using 60 or 120 seconds for the actual services)

Pre-Hooks can also be used to call HTTP endpoints.


#### deploy.yaml --example

``` yaml
apiVersion: v1
kind: Namespace
metadata:
  name: ing-test-ns
---
kind: ConfigMap
apiVersion: v1
metadata:
  name: fcubs-configmap
  namespace: ing-test-ns
data:
  index.html: |
    <html>
    <head><title>Hello FCUBS!</title></head>
    <body>
      <div>Hello FCUBS!</div>
    </body>
    </html>
---
apiVersion: v1
kind: Service
metadata:
  name: fcubs-service
  namespace: ing-test-ns
  labels:
    run: fcubs
spec:
  ports:
    - port: 80
      protocol: TCP
  selector:
    app:  fcubs
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: fcubs-deployment
  namespace: ing-test-ns
spec:
  selector:
    matchLabels:
      app: fcubs
  replicas: 1
  template:
    metadata:
      labels:
        app: fcubs
    spec:
      containers:
      - name: fcubs
        image: nginx:latest
        lifecycle:
          preStop:
            exec:
              command: [ "/bin/sleep", "20" ]
        ports:
        - containerPort: 80
        volumeMounts:
        - name: fcubs-volume
          mountPath: /usr/share/nginx/html/fcubs
      terminationGracePeriodSeconds: 60
      volumes:
      - name: fcubs-volume
        configMap:
          name: fcubs-configmap
```

#### Apply

```
¿ kubectl apply -f deploy.yaml
namespace/ing-test-ns created
configmap/fcubs-configmap created
service/fcubs-service created
deployment.apps/fcubs-deployment created

```

#### Check

```
kubectl get po -n ing-test-ns
NAME                                READY   STATUS    RESTARTS   AGE
fcubs-deployment-7bf4b58d6c-6scf7   1/1     Running   0          33s
```

Now lets delete the pod forcefully

```
kubectl delete po fcubs-deployment-7bf4b58d6c-6scf7 -n ing-test-ns
pod "fcubs-deployment-7bf4b58d6c-6scf7" deleted
```

You would notice that it would take 20 seconds to actually delete the pod


Lets use rollout restart and watch the pod using -w command

```
kubectl rollout restart deployment.apps/fcubs-deployment -n ing-test-ns && kubectl get po -n ing-test-ns -w
deployment.apps/fcubs-deployment restarted
NAME                                READY   STATUS              RESTARTS   AGE
fcubs-deployment-686477f67-s9tmz    0/1     ContainerCreating   0          0s
fcubs-deployment-7bf4b58d6c-nmtmm   1/1     Running             0          39s
fcubs-deployment-686477f67-s9tmz    1/1     Running             0          5s
fcubs-deployment-7bf4b58d6c-nmtmm   1/1     Terminating         0          44s
fcubs-deployment-7bf4b58d6c-nmtmm   0/1     Terminating         0          65s
fcubs-deployment-7bf4b58d6c-nmtmm   0/1     Terminating         0          65s
fcubs-deployment-7bf4b58d6c-nmtmm   0/1     Terminating         0          65s
^C%                                                                                               

kubectl get po -n ing-test-ns
NAME                               READY   STATUS    RESTARTS   AGE
fcubs-deployment-686477f67-s9tmz   1/1     Running   0          34s
```

You would notice that after the pod is moved to Terminating status at 44th second (fcubs-deployment-7bf4b58d6c-nmtmm 1/1 Terminating 0 44s), the next line (fcubs-deployment-7bf4b58d6c-nmtmm 0/1 Terminating 0 65s) comes at 65th second, which is roughly 20 sec later

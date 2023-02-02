    Created by Ranjith Vijayan, last modified on Aug 03, 2022

In this article, we will look at how to automate the "refreshing" of a deployment in Kubernetes using the Cronjob feature. Normally, the kubectl rollout restart command is used for this purpose, but it has to be done from outside the cluster. By using a Cronjob, we can schedule a Kubernetes API command to restart the deployment within the cluster.

To do this, we must set up RBAC (Role-Based Access Control) so that the Kubernetes client running inside the cluster has permission to make the necessary calls to the API. The Cronjob will use the bitnami/kubectl image, which is a kubectl binary that runs inside a container.

In this example, we are using a ClusterRole so that a single service account can run Kubernetes API for resources in any namespace. Alternatively, you can use a Role if you want to run this within a single namespace. The shell script that contains the kubectl rollout commands can be extended to add more business logic if needed. The schedule of the Cronjob can be controlled using the cron definition. This example runs the job every minute for illustration purposes only and is not practical. The schedule can be changed to run every day at midnight or every Sunday, for example.

Example:
The example creates a namespace called "util-restart-ns" for the service account, ClusterRole, and Cronjob. The deployments are created in a different namespace (ing-test-ns), and a RoleBinding object must be created in this namespace to allow it to use the RBAC service account.

1. deploy.yaml - to create deployment in "ing-test-ns"

deploy.yaml

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


2. restart.yaml. This will create the namespace "util-restart-ns" and the required objects in it.

restart.yaml

``` yaml

apiVersion: v1
kind: Namespace
metadata:
  name: util-restart-ns
---
kind: ServiceAccount
apiVersion: v1
metadata:
  name: deployment-restart
  namespace: util-restart-ns
---
# allow getting status and patching only the one deployment you want
# to restart
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: deployment-restart
  namespace: util-restart-ns
rules:
  - apiGroups: ["apps", "extensions"]
    resources: ["deployments"]
    verbs: ["get", "patch", "list", "watch"]
---
# bind the role to the service account
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: deployment-restart
  namespace: ing-test-ns
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: deployment-restart
subjects:
  - kind: ServiceAccount
    name: deployment-restart
    namespace: util-restart-ns

```



3. A shell script restart_deploy.sh that will be scheduled using cronjob

You can include as many deployments into this file, and include "business logic" if any. Its recommended to include "kubectl rollout status" for each rollout, to ensure rollouts commands are issued gradually to restart each deployment.

  restart_deploy.sh
``` bash
  kubectl rollout restart deployment/fcubs-deployment -n ing-test-ns
  kubectl rollout status deployment/fcubs-deployment -n ing-test-ns
```

4. load the above file as a configmap

```
  kubectl create configmap scripts-cm --from-file=restart_deploy.sh -n util-restart-ns
```

5. create the cronjob object. This will use the above script by attaching the configmap file

  cronjob.yaml

``` yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: deployment-restart
  namespace: util-restart-ns
spec:
  schedule: "* * * * *"
  #schedule: "@daily"
  jobTemplate:
    spec:
      template:
        spec:
          serviceAccountName: deployment-restart
          restartPolicy: Never
          containers:
          - name: kubectl
            image: bitnami/kubectl
            command: ["/bin/sh","/scripts/restart_deploy.sh"]
            volumeMounts:
            - name: scripts-vol
              mountPath: /scripts
          volumes:
          - name: scripts-vol
            configMap:
              name: scripts-cm

```

6. Testing

```
kubectl apply -f deploy.yaml
namespace/ing-test-ns created
configmap/fcubs-configmap created
service/fcubs-service created
deployment.apps/fcubs-deployment created


kubectl apply -f restart.yaml
namespace/util-restart-ns created
serviceaccount/deployment-restart created
clusterrole.rbac.authorization.k8s.io/deployment-restart created
rolebinding.rbac.authorization.k8s.io/deployment-restart created


kubectl create configmap scripts-cm --from-file=restart_deploy.sh -n util-restart-ns
configmap/scripts-cm created


kubectl apply -f cron.yaml
cronjob.batch/deployment-restart created


kubectl get po -n util-restart-ns -w
NAME                                READY   STATUS      RESTARTS   AGE
deployment-restart-27658708-ck7jq   0/1     Completed   0          13s
^C%                                                                                               


kubectl logs deployment-restart-27658708-ck7jq -n util-restart-ns
deployment.apps/fcubs-deployment restarted
Waiting for deployment spec update to be observed...
Waiting for deployment "fcubs-deployment" rollout to finish: 0 out of 1 new replicas have been updated...
Waiting for deployment "fcubs-deployment" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "fcubs-deployment" rollout to finish: 1 old replicas are pending termination...
deployment "fcubs-deployment" successfully rolled out


kubectl get po -n ing-test-ns -w
NAME                                READY   STATUS    RESTARTS   AGE
fcubs-deployment-558db7c796-xxdwv   1/1     Running   0          40s
fcubs-deployment-779558cd69-mgszk   0/1     Pending   0          0s
fcubs-deployment-779558cd69-mgszk   0/1     Pending   0          0s
fcubs-deployment-779558cd69-mgszk   0/1     ContainerCreating   0          0s
fcubs-deployment-779558cd69-mgszk   1/1     Running             0          4s
fcubs-deployment-558db7c796-xxdwv   1/1     Terminating         0          64s
fcubs-deployment-558db7c796-xxdwv   0/1     Terminating         0          85s
fcubs-deployment-558db7c796-xxdwv   0/1     Terminating         0          85s
fcubs-deployment-558db7c796-xxdwv   0/1     Terminating         0          85s

```

You can watch the pods of ing-test-ns using the last command, to see the pods are restarted in about 1 minute

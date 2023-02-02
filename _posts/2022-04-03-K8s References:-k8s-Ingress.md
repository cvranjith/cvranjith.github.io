    Created by Ranjith Vijayan on Apr 03, 2022

This post is created for my own reference of k8s resources as part of learning


Ingress exposes HTTP and HTTPS routes from outside the cluster to services within the cluster. Traffic routing is controlled by rules defined on the Ingress resource.


Refer to https://kubernetes.io/docs/concepts/services-networking/ingress/ for more details


Here is a simple example where an Ingress sends all its traffic to one Service:




In this example we are going to create a sample Ingress resource and map them to 2 services (called "fcubs" and "obpm") based on path rule.


Below yaml will create

* a namespace called ingress-test-ns
* 2 configmaps that will be mounted to the respective pods (i.e. fcubs & obpm)
* 2 services with corresponding deployment
*  an ingress resource with path based rule.


### ingress.yaml

``` yaml

apiVersion: v1
kind: Namespace
metadata:
  name: ingress-test-ns
---
kind: ConfigMap
apiVersion: v1
metadata:
  name: fcubs-configmap
  namespace: ingress-test-ns
data:
  index.html: |
    <html>
    <head><title>Hello FCUBS!</title></head>
    <body>
      <div>Hello FCUBS!</div>
    </body>
    </html>
---
kind: ConfigMap
apiVersion: v1
metadata:
  name: obpm-configmap
  namespace: ingress-test-ns
data:
  index.html: |
    <html>
    <head><title>Hello OBPM!</title></head>
    <body>
      <div>Hello OBPM!</div>
    </body>
    </html>
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-test
  namespace: ingress-test-ns
  annotations:
    ingress.kubernetes.io/ssl-redirect: "false"
spec:
  rules:
  - http:
      paths:
      - path: /obpm
        pathType: Prefix
        backend:
          service:
            name: obpm-service
            port:
              number: 80
      - path: /fcubs
        pathType: Prefix
        backend:
          service:
            name: fcubs-service
            port:
              number: 80
---
apiVersion: v1
kind: Service
metadata:
  name: obpm-service
  namespace: ingress-test-ns
  labels:
    run: obpm
spec:
  ports:
    - port: 80
      protocol: TCP
  selector:
    app:  obpm
 
---
apiVersion: v1
kind: Service
metadata:
  name: fcubs-service
  namespace: ingress-test-ns
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
  name: obpm-deployment
  namespace: ingress-test-ns
spec:
  selector:
    matchLabels:
      app: obpm
  replicas: 10
  template:
    metadata:
      labels:
        app: obpm
    spec:
      containers:
      - name: obpm
        image: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - name: obpm-volume
          mountPath: /usr/share/nginx/html/obpm
      volumes:
      - name: obpm-volume
        configMap:
          name: obpm-configmap
 
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: fcubs-deployment
  namespace: ingress-test-ns
spec:
  selector:
    matchLabels:
      app: fcubs
  replicas: 10
  template:
    metadata:
      labels:
        app: fcubs
    spec:
      containers:
      - name: fcubs
        image: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - name: fcubs-volume
          mountPath: /usr/share/nginx/html/fcubs
      volumes:
      - name: fcubs-volume
        configMap:
          name: fcubs-configmap
```


### Apply the yaml

``` bash
kubectl apply -f ingress.yaml
```

### Check if the ingress is configured.


``` bash
kubectl get ingress -n ingress-test-ns
NAME           CLASS    HOSTS   ADDRESS                                      PORTS   AGE
ingress-test   <none>   *       100.76.132.1,100.76.138.176,100.76.138.177   80      13m
```

Now if you send a request to the ingress port (which is 80), then it will route to the respective service based on URL path


E.g. if you launch the URL host:80/fcubs - it will route to "fcubs" service and display the HTML page (which was mounted via index.html configmap)

if you change the URL path to /obpm/ on the same same ip:port, it will route the traffic to the respective service (which is "obpm" in this case)

ingress automatically enables HTTPS traffic

e.g.
```
curl -k https://100.76.138.177/fcubs/
<html>
<head><title>Hello FCUBS!</title></head>
<body>
  <div>Hello FCUBS!</div>
</body>
</html>
```

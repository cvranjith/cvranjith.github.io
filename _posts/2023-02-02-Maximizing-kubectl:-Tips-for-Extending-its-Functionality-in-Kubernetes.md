    Created by Ranjith Vijayan on Feb 02, 2023

kubectl is a user-friendly tool that makes it simple to create new commands that build upon its standard functionality. A common task when using kubectl is to list pods to check their status and see if everything is running. To achieve this, you can use the command "kubectl get pods." However, typing this command repeatedly can become tedious. If you are coming from the docker experience, you would have developed the muscle memory to type 'docker ps'.  But if you type 'kubectl ps' you would get somethingl ike this

```
kubectl ps
error: unknown command "ps" for "kubectl"

Did you mean this?
        logs
        cp
```

To achieve something like `docker ps` with `kubectl`, you can create a custom script named "kubectl-ps" in the $PATH. When kubectl encounters a command it doesn't recognize, it will search for a specific command in the PATH. In this case, it will look for the "kubectl-ps" command.

1. Create a file by name `"kubectl-ps"` in one of the directories of your $PATH

``` bash
#!/bin/bash

kubectl get pods "$@"
```

2. Now if you type `kubectl ps` it will execute your custom script

```

kubectl ps
No resources found in default namespace.

```

Since we have added "$@" in the custom script we can pass arguments 

e.g.

```
kubectl ps -n kube-system 
NAME                                      READY   STATUS      RESTARTS   AGE
coredns-597584b69b-m8wfv                  1/1     Running     0          7d8h
local-path-provisioner-79f67d76f8-lnq2j   1/1     Running     0          7d8h
metrics-server-5f9f776df5-gbmrv           1/1     Running     0          7d8h
helm-install-traefik-crd-tdpbh            0/1     Completed   0          7d8h
helm-install-traefik-nhnkn                0/1     Completed   1          7d8h
svclb-traefik-8b85b943-5f2nb              2/2     Running     0          7d8h
traefik-66c46d954f-7ft6n                  1/1     Running     0          7d8h
```

Another example.

```
kubectl ps -A                                                                                     ✔  default ○  @fsgbu-mum-1019 
NAMESPACE         NAME                                      READY   STATUS      RESTARTS   AGE
kube-system       coredns-597584b69b-m8wfv                  1/1     Running     0          7d8h
kube-system       local-path-provisioner-79f67d76f8-lnq2j   1/1     Running     0          7d8h
kube-system       metrics-server-5f9f776df5-gbmrv           1/1     Running     0          7d8h
kube-system       helm-install-traefik-crd-tdpbh            0/1     Completed   0          7d8h
kube-system       helm-install-traefik-nhnkn                0/1     Completed   1          7d8h
kube-system       svclb-traefik-8b85b943-5f2nb              2/2     Running     0          7d8h
kube-system       traefik-66c46d954f-7ft6n                  1/1     Running     0          7d8h
ingress-test-ns   obpm-deployment-77dcdf4c78-cf2rh          1/1     Running     0          7d8h
ingress-test-ns   obpm-deployment-77dcdf4c78-smtr9          1/1     Running     0          7d8h
ingress-test-ns   fcubs-deployment-7ccd4b66fb-4r7fh         1/1     Running     0          7d8h
ingress-test-ns   fcubs-deployment-7ccd4b66fb-z6xxz         1/1     Running     0          7d8h
```


Of course, this is a trivial example, but you get the idea. Using this extensibility feature, you can build useful, convenient utilities with kubectl, which can save a lot of time and effort.




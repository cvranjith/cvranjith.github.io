   Created by Ranjith Vijayan on Apr 02, 2022

This post is created for my own reference of k8s resources as part of learning


k8s CronJob object is like one line of a crontab (cron table) file. It runs a job periodically on a given schedule, written in Cron format.


A typical use case is to clean up the Log files from Log directories on a periodic basis


A sample manifest to delete log files is below

cron.yaml

``` yaml

apiVersion: batch/v1
kind: CronJob
metadata:
  name: demo-clean
  namespace: pv-test
spec:
  #schedule: "* * * * *"
  schedule: "@daily"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: demo-clean
            image: fsgbu-mum-128.snbomprshared1.gbucdsint02bom.oraclevcn.com:5000/utils/busybox:latest
            args:
            - /bin/sh
            - -c
            - date; whoami; echo "Deleting files older than 2 days " && find /logs -type f -iname "*.log" -mtime +2 -print -delete && echo "Deleting empty Dir" && find /logs -empty -type d -print -delete
            volumeMounts:
            - name: pv
              mountPath: "/logs"
 
          restartPolicy: OnFailure
          volumes:
          - name: pv
            persistentVolumeClaim:
              claimName: task-pv-claim

```

*  Create the resource

``` bash
    kubectl apply -f cron.yaml
```

* Check

``` bash
    kubectl get cronjob -n pv-test -w
    kubectl get po -n pv-test  -w
    NAME                  READY   STATUS      RESTARTS   AGE
    task-pv-pod           1/1     Running     0          16h
    app-with-extn         1/1     Running     0          11h
    db-promote--1-bjvzw   0/1     Completed   0          23m
    demo-clean-27482730--1-7kp64   0/1     Pending     0          0s
    demo-clean-27482730--1-7kp64   0/1     Pending     0          0s
    demo-clean-27482730--1-7kp64   0/1     ContainerCreating   0          0s
    demo-clean-27482730--1-7kp64   0/1     Completed           0          4s
    demo-clean-27482730--1-7kp64   0/1     Completed           0          4s
```

*  See Logs

``` bash
    kubectl logs -n pv-test demo-clean-27482730--1-7kp64
```

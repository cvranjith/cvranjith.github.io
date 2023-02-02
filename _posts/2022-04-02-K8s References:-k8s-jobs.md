

    Created by Ranjith Vijayan, last modified on Apr 03, 2022



A Job creates one or more Pods and will continue to retry execution of the Pods until a specified number of them successfully terminate. As pods successfully complete, the Job tracks the successful completions. When a specified number of successful completions is reached, the task (ie, Job) is complete. Deleting a Job will clean up the Pods it created. Suspending a Job will delete its active Pods until the Job is resumed again.


Use-case:

If you have promotion of DB units via container images and if you want to apply this via K8s deployment, then Job type resource will be ideal


Below is a sample manifest for "one time job"


#### one-time-job.yaml

``` yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: db-promote
  namespace: pv-test
spec:
  ttlSecondsAfterFinished: 86400
  template:
    spec:
      restartPolicy: Never
      containers:
        - name: demo-job
          image: fsgbu-mum-128.snbomprshared1.gbucdsint02bom.oraclevcn.com:5000/utils/busybox:latest
          args:
          - /bin/sh
          - -c
          - date; echo "Hello One Time Job"
```

Note: ttlSecondsAfterFinished: implies the number of seconds after which the job resource will be removed, after successful completion. this helps in auto-cleanup of finished jobs.


### Apply

``` bash
    kubectl apply -f one-time-job.yaml
```
### Check

``` bash
    kubectl get jobs -n pv-test
    NAME         COMPLETIONS   DURATION   AGE
    db-promote   1/1           4s         15m
     
    kubectl get po -n pv-test 
    NAME                  READY   STATUS      RESTARTS   AGE
    db-promote--1-rgpnl   0/1     Completed   0          15m
```

### Check logs

``` yaml
    kubectl logs -n pv-test db-promote--1-rgpnl
    Sun Apr  3 04:35:53 UTC 2022
    Hello One Time Job
```


#### Note: 
  After the one-job is executed, if you want to promote another version of the image (typically with different tag), then you will have to delete and re-apply.

  Another strategy is to use different job-name from each db-promotion, and in conjunction with ttlSecondsAfterFinished  property to auto-cleanup the finished jobs.



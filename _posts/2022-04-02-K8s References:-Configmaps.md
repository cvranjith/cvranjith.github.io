    Created by Ranjith Vijayan on Feb 02, 2023

This post is created for my own reference of k8s resources as part of learning


Configmap based on yaml declaration

    Create yaml file
        e.g.

``` yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: filebeat-yaml-config
  namespace: fcubs
data:
  filebeat.yml: |-
    filebeat.inputs:
    - type: log
      enabled: true
      paths:
        - /opt/logs/**
      multiline.pattern: '^[0-9]{4}-[0-9]{2}-[0-9]{2}'
      multiline.negate: true
      multiline.match: after
  
    output.logstash:
      hosts: ["${LOGSTASH_HOST}:${LOGSTASH_PORT}"]
 
---
```


create using yaml
```
    kubectl create -f cfg-map.yaml
```

Configmaps can be crated from file using --from-file argument. Multiple files in a directory can be supplied to configmap by supplying the directory name. This is useful for binary files.


In the below example the there are 2 files in "property" folder, and we can create configmap using this command
```
kubectl create configmap prop-keystore --from-file=property -n fcubs
```

To view the configmap. Please note in k8s dashboard the binary data won't be displayed. Use below command to view the binary data (in base64 format)

```
kubectl get cm prop-keystore -n fcubs -o json
```

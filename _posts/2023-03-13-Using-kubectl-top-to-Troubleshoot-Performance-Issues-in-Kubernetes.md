`kubectl top` is a command that allows you to get resource usage information for Kubernetes objects such as pods, nodes, and containers. With this command, you can monitor and troubleshoot the resource usage of your Kubernetes cluster.

In this blog post, we will focus on kubectl top commands for pods and nodes, and some examples of how to use these commands to sort the pods by CPU/memory usage.

*** kubectl top pod

The `kubectl top` pod command allows you to get CPU and memory usage information for pods running in your cluster. To use this command, simply run:

``` cpp
kubectl top pod
```

This will show the CPU and memory usage for all pods in the default namespace. If you want to view the usage for a specific namespace, you can use the -n option:

``` cpp
kubectl top pod -n <namespace>
```


To sort the pods by CPU or memory usage, you can use the --sort-by option. For example, to sort the pods by CPU usage, run:

``` css
kubectl top pod --sort-by cpu

```

This will show the pods sorted by CPU usage in descending order. To sort the pods by memory usage, run:

``` css
kubectl top pod --sort-by memory
```


*** kubectl top node

The kubectl top node command allows you to get CPU and memory usage information for nodes in your cluster. To use this command, simply run:

``` css
kubectl top node
```

This will show the CPU and memory usage for all nodes in your cluster. To view the usage for a specific node, you can use the -l option:

``` css
kubectl top node -l <node-name>
```

To sort the nodes by CPU or memory usage, you can use the --sort-by option. For example, to sort the nodes by CPU usage, run:

``` css
kubectl top node --sort-by cpu
```

This will show the nodes sorted by CPU usage in descending order. To sort the nodes by memory usage, run:

``` css
kubectl top node --sort-by memory
```

*** Other common useful features

Here are some other common features that you may find useful when using kubectl top:

To view the resource usage for a specific container in a pod, you can use the -c option followed by the container name:

``` php
kubectl top pod <pod-name> -c <container-name>
```

To view the resource usage for all containers in a pod, you can use the --containers option:

``` css
kubectl top pod <pod-name> --containers
```

To view the resource usage for a specific node over time, you can use the -w option followed by the node name:

``` css
kubectl top node -w -l <node-name>
```

This will show the resource usage for the node in real-time.

To view the top processes consuming CPU in a container, you can use the top command:

``` php
kubectl exec <pod-name> -c <container-name> -- top
```

This will show the top processes consuming CPU in the container.

In conclusion, kubectl top is a powerful command that allows you to monitor the resource usage of your Kubernetes cluster. With the ability to sort by CPU and memory usage, as well as other useful features, this command can help you troubleshoot performance issues and optimize resource allocation.

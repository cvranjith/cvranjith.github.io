

In the monolithic era of software, picture 8 developers working inside a large shared bungalow. This bungalow represents a monolithic application - one big, unified codebase where everyone operates under the same roof.

Inside the house are 10 power sockets - representing CPU cores. Each developer (i.e. a unit of workload) plugs in their laptop to just one socket. Standing at the entrance is a security guard - the quota enforcer - who monitors the power board to ensure the total socket usage doesn’t exceed the building’s capacity.

At any given time, only 8 of the 10 sockets are in use, and the guard sees all green lights on the power board. Everyone works happily - predictable usage, and no socket shortages.


Then came microservices - and with them, modernization.

Now, each developer is given their own studio apartment - representing a Kubernetes pod. These are cleaner, isolated, and individually managed. The same security guard has now been promoted to manage the entire apartment complex. He still tracks how many sockets are in use across all apartments.

As each developer moves in, the guard asks: “How many power sockets will you need?”

One replies: “I really only need 1 socket to work, but during setup I’ll plug in 2 temporarily - one for my laptop, one for configuration. Then I’ll scale back to 1”

The guard allows it and updates his tracker. This continues for the next few tenants.


But when the 6th person arrives and makes the same request, the guard raises a hand: “Stop - we’ve reached the limit. You can't move in”

Confused, the new tenant protests: “But no one’s using all their 2 sockets at the same time!”


## What Just Happened?


The guard is counting total declared limits, not actual usage. Even if the building has enough power physically available, the system blocks new tenants based on maximum requested limits, to stay within the defined quota. And just like that, a well-intentioned enforcement policy leads to underutilization and startup delays. 


This story is exactly what happens when Kubernetes enforces limits.cpu  quotas, not at runtime, but at deploy time. This is where Kubernetes overcommit architecture steps in - to bridge the gap between rigid quota enforcement and dynamic, efficient resource sharing

## What Is Overcommit Architecture?


The idea of overcommit comes from a practical truth:


Most applications do not use 100% of their requested resources all the time.


Traditionally, sizing applications has always been tricky. In mission-critical environments, the standard practice was to provision for the worst case - i.e. the peak traffic of the year, the highest load imaginable. This meant that:

    You sized your systems for maximum throughput
    You bought capacity for the spike, not the norm

And as a result, most of your capacity sat idle for most of the time. This over-provisioning led to massive resource wastage - CPU and memory that were paid for, but rarely used.

Cloud platforms flipped this model with one foundational idea:

Elasticity - scale up when you need more, scale down when you don't.

You no longer need to own all the capacity upfront. You can share it with others, and only pay for what you use.
But how do you achieve that technically?

This is where Kubernetes overcommit architecture shines. It introduces a clear distinction between:

    requests  – what your app is guaranteed to get (used for scheduling)
    limits  – the ceiling your app is allowed to reach (enforced at runtime)


By allowing limits across all workloads to exceed the actual cluster size, Kubernetes:

    packs workloads more efficiently
    Enables runtime bursts
    Reduces waste while honoring guarantees


This is not just a cost-saving trick. It’s the core philosophy behind microservices and cloud-native systems, where each component is small, efficient, and elastic by design.


Overcommit architecture is how this philosophy is implemented at scale - not just by Kubernetes, but by the cloud itself. It:

    Enables better resource utilization
    Supports elastic behavior (e.g., short bursts during startup)
    Avoids wasting capacity when workloads are idle


In other words:   Overcommit is how Kubernetes - and all modern cloud platforms - run economically.

## Why Some Customers Enforce Hard Limits

In some customer environments, especially in multi-tenant clusters, platform admins set quota with limits.cpu  and request.cpu with the same value.

E.g.
limits.cpu: 8
requests.cpu: 8


Their concern is valid- “We don’t want one app to hog all CPU. Let’s cap it at 8.”


But here’s the flaw: By setting the limit quota = request quota, they’re disabling overcommit entirely.

This:
- Blocks deployments when limits exceed quota - even if the app only uses 8 cores in practice
- Prevents startup bursts (like copying files or initializing caches)
- Wastes idle cluster capacity that could’ve been reused

## ✅ What would a right model look like

Lets take a look at OpenShift Developer Sandbox - a real-world multi-tenant Kubernetes environment.

Here’s what the quota looks like:

`oc describe quota`

```
requests.cpu: 3
limits.cpu:   30
```

This is smart. Because:

- `requests.cpu` is low → ensures fair scheduling
- `limits.cpu` is higher → allows bursting if capacity exists

To validate how Kubernetes handles resource evaluation, we ran a real test:

The Setup
```
initContainers:
  - 5 init containers @
       limit: 100m CPU
       request: 20m CPU
  - 1 init container
        limit: 150m CPU
       request: 20m CPU
  
main container:
        limit: 200m CPU
        request: 50m CPU
```

Kubernetes Quota Math:

Kubernetes evaluates the pod's resource usage like this:

* Effective CPU limit = max(init limits) vs sum(app limits) = max(200m, 150m) = 200m ✅
* Effective CPU request = max(init requests) vs sum(app requests) = max(20m, 50m) = 50m ✅


Confirmed with:

`oc describe quota`

Output:
```
limits.cpu:     200m / 30
requests.cpu:    50m / 3
```

Even though the pod had multiple init containers, only the highest single init container value was considered, not the sum - just as Kubernetes promises. This also proves that Kubernetes handles sequential init containers to optimize quota evaluation.

And importantly - this setup in the OpenShift sandbox, where the namespace has a quota of 30 cores for limits, does not mean each application is promised 30 cores. It simply means the cluster allows tenants to burst up to that total. These limits are enforced at the cluster level, not reserved per application. So you don't statically allocate 30 cores per app - it’s a shared ceiling.


If the cluster doesn’t have spare capacity at runtime, Kubernetes will throttle workloads that exceed their requests... And that’s fair - because requests are guaranteed, but limits are not promises.


However, if the cluster does have spare capacity, Kubernetes will allow workloads to burst, based on a first-come, first-served basis. This is an elegant trade-off between efficiency and fairness in a multi-tenant model. Also, this model supports monitoring-driven feedback. If an application consistently bursts above its baseline requests, tools like Prometheus can trigger a sizing review - helping teams adjust resource configurations to match the usage.

This proves how Kubernetes strikes a balance: predictability for scheduling, flexibility for efficiency, and observability for accountability

## Conclusion

Overcommit is not a workaround - it’s how cloud platforms work. Just like in the apartment analogy, you're not installing a permanent second plug - you're just asking to use it if no one else is. And Kubernetes, like a smart building manager, will let you - unless the building's full. Let Kubernetes do what it was built to do: allocate wisely, share efficiently, and scale responsibly.




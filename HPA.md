**HPA = Horizontal Pod Autoscaler** in Kubernetes.

It automatically **increases or decreases the number of Pod replicas** based on resource usage or other metrics.

### Example

Suppose your application normally has:

```text
2 Pods
```

If CPU usage becomes high:

```text
2 Pods → 3 Pods → 5 Pods
```

When traffic decreases:

```text
5 Pods → 3 Pods → 2 Pods
```

### Why `resource requests/limits` with HPA?

HPA commonly uses CPU or memory utilization. Kubernetes needs **resource requests** to calculate utilization correctly.

```yaml
resources:
  requests:
    cpu: "200m"
    memory: "256Mi"
  limits:
    cpu: "500m"
    memory: "512Mi"
```

For example:

* Pod CPU request = `200m`
* Current CPU usage = `160m`
* Usage = **80%**
* If HPA target is 70%, Kubernetes may create more Pods.

So:

> **HPA = automatically scales the number of Pods horizontally based on demand.**

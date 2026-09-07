Yes, absolutely. **You should refer to the official Kubernetes documentation**, especially because you're new to Kubernetes and your assignment will have a thorough code review. Official docs are the best source to learn what each resource actually does.

First, don't feel overwhelmed by this folder:

```text
k8s/
├── deployment.yaml
├── service.yaml
├── gateway.yaml
├── httproute.yaml
├── hpa.yaml
├── pdb.yaml
├── configmap.yaml
└── secret.yaml
```

These are just **different Kubernetes configuration files**. Each file tells Kubernetes how to manage one aspect of your application.

---

# Think about your BloomWorld backend

Your application currently works like this:

```text
Browser
   ↓
Shop Service
   ↓
PostgreSQL
```

In Kubernetes, it becomes:

```text
Internet
   ↓
External Load Balancer
   ↓
Gateway
   ↓
HTTPRoute
   ↓
Service
   ↓
Deployment
   ↓
Pods (your Shop Service containers)
```

And additional files help with:

```text
Scaling → HPA
Availability → PDB
Configuration → ConfigMap
Secrets → Secret / Secret Manager
```

---

# 1. `deployment.yaml` — Runs your application

This is one of the most important files.

It tells Kubernetes:

> "Run my Shop Service container."

For example:

```text
Deployment
    │
    ├── Pod 1 → Shop Service Container
    │
    └── Pod 2 → Shop Service Container
```

It can control:

* Number of replicas
* Docker image
* CPU/memory resources
* Health checks
* Rolling updates

For your assignment, this is **essential**.

Official documentation: [Kubernetes Deployments documentation](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/?utm_source=chatgpt.com)

---

# 2. `service.yaml` — Gives Pods a stable network endpoint

Pods can be recreated and their IP addresses can change.

A Service provides a stable way to reach your Pods:

```text
Gateway
   ↓
Service
   ↓
Pod 1
Pod 2
```

For your backend, we will probably use:

```text
ClusterIP
```

Because your Shop Service Pods should not be directly exposed to the internet.

Official documentation: [Kubernetes Services documentation](https://kubernetes.io/docs/concepts/services-networking/service/?utm_source=chatgpt.com)

---

# 3. `gateway.yaml` — Controls incoming traffic

Your assignment specifically requires a **Gateway**.

Think of it as the Kubernetes traffic entry point:

```text
Internet
   ↓
External Load Balancer
   ↓
Gateway
```

The Gateway API provides advanced traffic routing and can represent traffic infrastructure such as a cloud load balancer. ([Kubernetes][1])

Official documentation: [Kubernetes Gateway API documentation](https://kubernetes.io/docs/concepts/services-networking/gateway/?utm_source=chatgpt.com)

---

# 4. `httproute.yaml` — Decides where HTTP traffic goes

The Gateway receives traffic.

But Kubernetes needs to know:

> "Where should `/api` requests go?"

That's the job of `HTTPRoute`.

Example:

```text
Request:

/api/shops

       ↓

HTTPRoute

       ↓

shop-service
```

Official docs describe `HTTPRoute` as defining HTTP routing behavior from a Gateway to backend endpoints such as a Kubernetes Service. ([Kubernetes][1])

---

# 5. `hpa.yaml` — Automatically scales your Pods

HPA means:

> **Horizontal Pod Autoscaler**

Suppose your application normally has:

```text
2 Pods
```

Then many users visit your application:

```text
High CPU
    ↓
HPA
    ↓
4 Pods
```

When traffic decreases:

```text
Low CPU
    ↓
HPA
    ↓
2 Pods
```

Kubernetes HPA automatically adjusts scalable workloads such as Deployments based on observed metrics such as CPU or memory. ([Kubernetes][2])

Official documentation: [Horizontal Pod Autoscaling documentation](https://kubernetes.io/docs/concepts/workloads/autoscaling/horizontal-pod-autoscale/?utm_source=chatgpt.com)

---

# 6. `pdb.yaml` — Protects availability during disruptions

PDB means:

> **Pod Disruption Budget**

Imagine you have:

```text
2 Pods

Pod 1 ✅
Pod 2 ✅
```

During node maintenance, Kubernetes may need to remove Pods.

Without protection:

```text
Pod 1 ❌
Pod 2 ❌

Application unavailable 😨
```

With a PDB, you can tell Kubernetes:

> "Always keep at least one application Pod available."

```text
2 Pods
   ↓
Node maintenance
   ↓
1 Pod can be disrupted
   ↓
1 Pod stays running ✅
```

A PDB limits voluntary disruptions to a collection of Pods. ([Kubernetes][3])

Official documentation: [Pod Disruptions and PodDisruptionBudgets documentation](https://kubernetes.io/docs/concepts/workloads/pods/disruptions/?utm_source=chatgpt.com)

---

# 7. `configmap.yaml` — Stores non-secret configuration

For example:

```text
NODE_ENV=production
PORT=3000
```

These are configuration values that are **not sensitive**.

Instead of hardcoding them in your application:

```js
const PORT = 3000;
```

you can inject configuration into your container.

```text
ConfigMap
     ↓
Environment Variables
     ↓
Shop Service Pod
```

Official documentation: [Kubernetes ConfigMaps documentation](https://kubernetes.io/docs/concepts/configuration/configmap/?utm_source=chatgpt.com)

---

# 8. `secret.yaml` — Stores sensitive configuration

For example:

```text
Database password
Database username
API keys
```

You should not put these directly inside:

```text
deployment.yaml ❌
```

Instead:

```text
Secret
   ↓
Deployment
   ↓
Container environment variables
```

Official documentation: [Kubernetes Secrets documentation](https://kubernetes.io/docs/concepts/configuration/secret/?utm_source=chatgpt.com)

⚠️ For your GCP assignment, we may improve this further by using:

```text
Google Secret Manager
```

instead of storing important production secrets directly in a Kubernetes YAML file.

---

# The most important thing for you

You **do not need to learn all of these at once**.

We should learn them in this order:

```text
STEP 1
Deployment
   ↓
STEP 2
Service
   ↓
STEP 3
ConfigMap + Secret
   ↓
STEP 4
Gateway + HTTPRoute
   ↓
STEP 5
HPA
   ↓
STEP 6
PDB
```

So first, you only need to understand:

```text
Deployment → runs Pods
Service → connects to Pods
```

Once you understand those, the rest becomes much easier.

### My recommendation

Before writing any Kubernetes YAML, let's first understand **Deployment → Pod → Container** using your existing `shop-service`, because that's the foundation for everything else.

[1]: https://kubernetes.io/docs/concepts/services-networking/gateway/?utm_source=chatgpt.com "Gateway API | Kubernetes"
[2]: https://kubernetes.io/docs/concepts/workloads/autoscaling/horizontal-pod-autoscale/?utm_source=chatgpt.com "Horizontal Pod Autoscaling | Kubernetes"
[3]: https://kubernetes.io/docs/reference/kubernetes-api/policy/pod-disruption-budget-v1/?utm_source=chatgpt.com "PodDisruptionBudget | Kubernetes"

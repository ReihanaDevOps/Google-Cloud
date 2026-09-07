This assignment is expecting you to act like a **DevOps Engineer designing and deploying a production-style application infrastructure**.

The main focus is **not building a complex application**. The main focus is:

> **Can you design, provision, deploy, automate, secure, and operate an application on cloud infrastructure?**

Let's break down exactly what they expect.

---

# 1. Application architecture

They want something like this:

```text
                    Internet
                       │
                       ▼
              Static Landing Page
              (Outside Kubernetes)
                       │
                       │ API Calls
                       ▼
                External Gateway
                       │
                       ▼
              Backend Application
              (Kubernetes Pods)
                       │
                       ▼
                   Database
```

You need:

### Frontend

```text
Static Landing Page / SPA
```

Hosted **outside Kubernetes**.

For example on GCP:

```text
Cloud Storage
     +
Cloud CDN
```

---

### Backend

One backend application:

```text
Node.js + Express
```

Deployed into:

```text
Kubernetes Cluster
```

You are currently building:

```text
shop-service
```

That is enough.

---

# 2. Infrastructure using Terraform

They expect you to create cloud infrastructure using Terraform instead of manually clicking everything in the cloud console.

For example:

```text
Terraform
   │
   ├── Network
   │     ├── VPC
   │     └── Subnets
   │
   ├── Kubernetes Cluster
   │
   ├── Node Pools
   │
   ├── Secret Management
   │
   ├── Storage
   │
   └── Database
```

For GCP, this could be:

```text
Terraform
   │
   ├── VPC
   ├── Subnet
   ├── GKE Cluster
   ├── Node Pool
   ├── Secret Manager
   ├── Cloud SQL
   └── Cloud Storage
```

---

# 3. Kubernetes best practices

This is a major part of the assignment.

They don't just want:

```text
kubectl apply deployment.yaml
```

They want you to consider production practices.

Your backend deployment should have:

### Multiple replicas

```text
Shop Service
├── Pod 1
├── Pod 2
└── Pod 3
```

For example:

```yaml
replicas: 2
```

This gives availability.

If one pod dies:

```text
Pod 1 ❌

Pod 2 ✅ Still running
```

---

### Resource requests and limits

Example:

```yaml
resources:
  requests:
    cpu: "100m"
    memory: "128Mi"

  limits:
    cpu: "500m"
    memory: "512Mi"
```

This demonstrates:

```text
Efficient resource utilization
```

which they specifically mentioned.

---

### Autoscaling

Use:

```text
Horizontal Pod Autoscaler
```

For example:

```text
Normal Traffic
     │
     ▼
   2 Pods

High Traffic
     │
     ▼
   5 Pods
```

Based on:

```text
CPU
Memory
```

---

### Resilience during updates

Use:

```text
RollingUpdate
```

Example:

```text
Version 1 Pods
     │
Deploy new version
     │
     ▼
Version 2 Pods gradually replace Version 1
```

Without downtime.

Also potentially:

```text
PodDisruptionBudget
```

to help during node disruptions.

---

# 4. Gateway traffic management

They explicitly said:

> Use Gateway to manage traffic.

So your architecture should include:

```text
Internet
   │
External Load Balancer
   │
Gateway
   │
   ▼
Backend Service
   │
   ▼
Pods
```

In GCP, we can use:

```text
GKE Gateway API
```

The Gateway manages incoming traffic and routes it to your Kubernetes Service.

For example:

```text
/api/*
   │
   ▼
Gateway
   │
   ▼
shop-service
```

---

# 5. Database

They expect:

> a small database

This does **not need to be a complicated database architecture**.

For our project:

```text
Shop Service
      │
      ▼
PostgreSQL
```

On GCP, the better production approach would be:

```text
Cloud SQL (PostgreSQL)
```

instead of running PostgreSQL inside Kubernetes.

Why?

Because:

```text
Application → Kubernetes
Database → Managed Cloud Service
```

This is usually easier to manage and demonstrates good cloud architecture.

So we could use:

```text
GKE
 │
 └── Shop Service Pods

Cloud SQL
 │
 └── PostgreSQL
```

---

# 6. Secret management

They don't want this:

```javascript
const password = "bloomworld123";
```

inside your code. ❌

They want:

```text
Google Secret Manager
```

Store things like:

```text
DATABASE_PASSWORD
DATABASE_CONNECTION_STRING
API_KEYS
```

Then:

```text
Google Secret Manager
        │
        ▼
Kubernetes Application
```

This demonstrates security best practices.

---

# 7. Static Landing Page outside Kubernetes

This is important.

They specifically said:

> Static Landing Page Hosting (outside the cluster)

So your React frontend should **not necessarily be deployed as a Kubernetes Pod**.

Instead:

```text
React Build
    │
    ▼
Static Files
    │
    ▼
Cloud Storage
    │
    ▼
Cloud CDN
```

For GCP:

```text
Cloud Storage
+
Cloud CDN
```

Benefits:

```text
Cheap
Fast
Scalable
```

This directly answers their requirement.

---

# 8. CI/CD Pipeline

They want automation.

Something like:

```text
Developer
    │
    ▼
GitHub Push
    │
    ▼
GitHub Actions
    │
    ├── Install Dependencies
    │
    ├── Run Tests
    │
    ├── Build Application
    │
    ├── Build Docker Image
    │
    ├── Push Image
    │
    └── Deploy to Kubernetes
```

Example:

```text
GitHub
   │
   ▼
GitHub Actions
   │
   ▼
Docker Build
   │
   ▼
Artifact Registry
   │
   ▼
GKE Deployment
```

They specifically mention:

### Optimize Dockerfile

They may expect things like:

```text
Multi-stage build
Small base image
.dockerignore
```

Example concept:

```text
Build Stage
     │
     ▼
Production Image
```

Not a huge Docker image containing unnecessary files.

---

# 9. Bonus: ArgoCD

If you want extra points:

```text
GitHub
   │
   ▼
ArgoCD
   │
   ▼
GKE
```

Instead of CI/CD directly running:

```text
kubectl apply
```

You use:

```text
Git Repository
       │
       │ Desired Kubernetes State
       ▼
ArgoCD
       │
       ▼
Kubernetes Cluster
```

This is called:

> GitOps

But this is **bonus**, so don't make the core project depend on it.

---

# 10. Monitoring and Governance — Bonus

They mention:

### Governance

Examples:

```text
IAM
RBAC
Resource Quotas
Network Policies
Organization Policies
```

### Monitoring

For GCP:

```text
Cloud Monitoring
Cloud Logging
```

Or:

```text
Prometheus
Grafana
```

You could monitor:

```text
CPU usage
Memory usage
Pod health
Application errors
Request latency
```

---

# 🎯 What I think you should build

For your assignment, I recommend this architecture:

```text
                     INTERNET
                         │
              ┌──────────┴──────────┐
              │                     │
              ▼                     ▼
       Static React SPA         GKE Gateway
       Cloud Storage                │
       + Cloud CDN                  │
                                   ▼
                         External Load Balancer
                                   │
                                   ▼
                            Gateway API
                                   │
                                   ▼
                         Shop Service (Node.js)
                                   │
                         ┌─────────┴─────────┐
                         ▼                   ▼
                      Pod 1               Pod 2
                         │                   │
                         └─────────┬─────────┘
                                   │
                                   ▼
                         Cloud SQL PostgreSQL


Infrastructure:
Terraform → VPC → GKE → Cloud SQL → Secret Manager → Storage

CI/CD:
GitHub → GitHub Actions → Artifact Registry → GKE
```

---

# What the assignment is REALLY testing

| Area         | What they want to see                      |
| ------------ | ------------------------------------------ |
| Terraform    | Can you automate infrastructure?           |
| Kubernetes   | Can you deploy production-style workloads? |
| Networking   | Can you configure traffic correctly?       |
| Gateway      | Can you manage application traffic?        |
| Database     | Can your application use cloud services?   |
| Security     | Can you handle secrets properly?           |
| Docker       | Can you containerize efficiently?          |
| CI/CD        | Can you automate deployments?              |
| Availability | Can your application survive failures?     |
| Scaling      | Can it scale under load?                   |
| Cost         | Can you choose efficient services?         |

---

## My recommendation for our project

We should **keep the application simple**:

```text
1 React SPA
+
1 Node.js Backend
+
1 PostgreSQL Database
```

Then put most of our effort into the **DevOps architecture**, because that is what this assignment is actually evaluating.

So yes, our **one `shop-service` backend is correct**. We don't need multiple microservices.

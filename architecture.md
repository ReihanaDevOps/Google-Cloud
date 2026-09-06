                    Internet
                       │
                       ▼
              External Load Balancer
                       │
                       ▼
                  Gateway
                       │
            ┌──────────┴──────────┐
            │                     │
            ▼                     ▼
      Shop Service           Order Service
         Pods                   Pods
            │                     │
            └──────────┬──────────┘
                       │
                    Database
<h2>Deploy your backend on GKE, expose it through a Kubernetes Gateway, and use an external load balancer to receive internet traffic.</h2>

bloomworld/
│
├── frontend/          → React SPA
├── shop-service/      → Microservice 1
├── order-service/     → Microservice 2
├── kubernetes/        → Kubernetes YAML files
└── terraform/         → Infrastructure code

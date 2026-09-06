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

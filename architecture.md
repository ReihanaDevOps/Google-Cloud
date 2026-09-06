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

<img width="787" height="270" alt="image" src="https://github.com/user-attachments/assets/f67f74bf-837a-468b-a131-2e6cf90e1a48" />

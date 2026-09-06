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

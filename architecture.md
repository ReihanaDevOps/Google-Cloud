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

<img width="336" height="160" alt="image" src="https://github.com/user-attachments/assets/5056c65c-bd59-4ba1-a954-47a81ca17187" />

React SPA
    │
    │ HTTP Request
    ▼
Shop Service (Node.js + Express)
    │
    ├── Shops
    ├── Flowers
    ├── Custom Bouquets
    └── Orders



Client
  │
  │ GET /api/shops
  ▼
Express App
  │
  ▼
Shop Routes
  │
  ▼
Shop Controller
  │
  ▼
Temporary Shop Data

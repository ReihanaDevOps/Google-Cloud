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


{
  "name": "Bloom Kandy",
  "city": "Kandy"
}


<img width="835" height="577" alt="image" src="https://github.com/user-attachments/assets/bb8a3c1c-c7ec-457e-81c1-c9abb8e470a9" />


1. Push BloomWorld → GitHub
        ↓
2. Create High-Level + Low-Level Architecture diagrams
        ↓
3. Write Terraform
        ↓
4. Provision infrastructure in GCP
        ↓
   VPC
   GKE
   Cloud SQL
   Secret Manager
   Artifact Registry
   Cloud Storage
        ↓
5. Create Kubernetes manifests
        ↓
6. Test deploy Shop Service manually to GKE
        ↓
7. Set up Jenkins
        ↓
8. Create Jenkins CI/CD Pipeline
        ↓
   Build
   → Test
   → Docker Build
   → Push Artifact Registry
   → Deploy to GKE
        ↓
9. Monitoring / Optional Bonuses

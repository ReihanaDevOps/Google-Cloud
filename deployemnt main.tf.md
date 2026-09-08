Yes. Based on the infrastructure we discussed, here is a **single complete `main.tf`** containing the main resources.

This is a **learning/starter configuration**, not yet a fully production-hardened setup.

## `main.tf`

```hcl
# ============================================================
# ENABLE REQUIRED GCP APIs
# ============================================================

resource "google_project_service" "compute" {
  service            = "compute.googleapis.com"
  disable_on_destroy = false
}

resource "google_project_service" "container" {
  service            = "container.googleapis.com"
  disable_on_destroy = false
}

resource "google_project_service" "artifact_registry" {
  service            = "artifactregistry.googleapis.com"
  disable_on_destroy = false
}

resource "google_project_service" "sqladmin" {
  service            = "sqladmin.googleapis.com"
  disable_on_destroy = false
}

resource "google_project_service" "secretmanager" {
  service            = "secretmanager.googleapis.com"
  disable_on_destroy = false
}


# ============================================================
# VPC
# ============================================================

resource "google_compute_network" "bloomworld_vpc" {
  name                    = "${var.environment}-bloomworld-vpc"
  auto_create_subnetworks = false

  depends_on = [
    google_project_service.compute
  ]
}


# ============================================================
# GKE SUBNET
# ============================================================

resource "google_compute_subnetwork" "gke_subnet" {
  name          = "${var.environment}-gke-subnet"
  region        = var.region
  network       = google_compute_network.bloomworld_vpc.id
  ip_cidr_range = "10.10.0.0/16"

  # IP range for Kubernetes Pods
  secondary_ip_range {
    range_name    = "pods"
    ip_cidr_range = "10.20.0.0/16"
  }

  # IP range for Kubernetes Services
  secondary_ip_range {
    range_name    = "services"
    ip_cidr_range = "10.30.0.0/20"
  }
}


# ============================================================
# IAM - GKE NODE SERVICE ACCOUNT
# ============================================================

resource "google_service_account" "gke_nodes" {
  account_id   = "${var.environment}-gke-nodes"
  display_name = "GKE Node Service Account"
}


# Required permissions for GKE nodes

resource "google_project_iam_member" "gke_nodes_default" {
  project = var.project_id

  role = "roles/container.defaultNodeServiceAccount"

  member = "serviceAccount:${google_service_account.gke_nodes.email}"
}


# Allow GKE nodes to pull images from Artifact Registry

resource "google_project_iam_member" "artifact_registry_reader" {
  project = var.project_id

  role = "roles/artifactregistry.reader"

  member = "serviceAccount:${google_service_account.gke_nodes.email}"
}


# ============================================================
# ARTIFACT REGISTRY
# ============================================================

resource "google_artifact_registry_repository" "bloomworld" {
  location      = var.region
  repository_id = "${var.environment}-bloomworld"

  description = "Docker repository for BloomWorld"

  format = "DOCKER"

  depends_on = [
    google_project_service.artifact_registry
  ]
}


# ============================================================
# GKE CLUSTER
# ============================================================

resource "google_container_cluster" "bloomworld" {
  name     = var.cluster_name
  location = var.region

  network    = google_compute_network.bloomworld_vpc.id
  subnetwork = google_compute_subnetwork.gke_subnet.id

  deletion_protection = false

  # Remove default node pool
  remove_default_node_pool = true
  initial_node_count       = 1

  # VPC-native networking
  networking_mode = "VPC_NATIVE"

  ip_allocation_policy {
    cluster_secondary_range_name  = "pods"
    services_secondary_range_name = "services"
  }

  # Workload Identity
  workload_identity_config {
    workload_pool = "${var.project_id}.svc.id.goog"
  }

  release_channel {
    channel = "REGULAR"
  }

  depends_on = [
    google_project_service.container
  ]
}


# ============================================================
# GKE NODE POOL
# ============================================================

resource "google_container_node_pool" "primary_nodes" {
  name     = "${var.environment}-node-pool"
  location = var.region

  cluster = google_container_cluster.bloomworld.name

  node_count = 1

  node_config {
    machine_type = "e2-medium"

    service_account = google_service_account.gke_nodes.email

    oauth_scopes = [
      "https://www.googleapis.com/auth/cloud-platform"
    ]

    labels = {
      environment = var.environment
    }
  }

  management {
    auto_repair  = true
    auto_upgrade = true
  }

  depends_on = [
    google_project_iam_member.gke_nodes_default
  ]
}


# ============================================================
# CLOUD SQL - POSTGRESQL
# ============================================================

resource "google_sql_database_instance" "bloomworld" {
  name             = "${var.environment}-bloomworld-db"
  database_version = "POSTGRES_16"

  region = var.region

  settings {
    tier = "db-f1-micro"
  }

  deletion_protection = false

  depends_on = [
    google_project_service.sqladmin
  ]
}


# Create application database

resource "google_sql_database" "bloomworld_database" {
  name     = "bloomworld"

  instance = google_sql_database_instance.bloomworld.name
}


# ============================================================
# SECRET MANAGER
# ============================================================

resource "google_secret_manager_secret" "db_password" {
  secret_id = "${var.environment}-db-password"

  replication {
    auto {}
  }

  depends_on = [
    google_project_service.secretmanager
  ]
}
```

---

## Your folder now

```text
terraform/
│
├── provider.tf
├── variables.tf
├── main.tf       ← All infrastructure resources
└── outputs.tf
```

### `provider.tf`

```hcl
terraform {
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "~> 7.0"
    }
  }
}

provider "google" {
  project = var.project_id
  region  = var.region
}
```

### `variables.tf`

```hcl
variable "project_id" {
  type = string
}

variable "region" {
  type    = string
  default = "asia-south1"
}

variable "cluster_name" {
  type    = string
  default = "bloomworld-gke"
}

variable "environment" {
  type    = string
  default = "dev"
}
```

## Architecture this creates

```text
GCP Project
│
├── VPC
│    └── GKE Subnet
│         ├── Nodes
│         ├── Pods
│         └── Services
│
├── GKE Cluster
│    └── Node Pool
│
├── IAM
│    └── GKE Node Service Account
│
├── Artifact Registry
│
├── Cloud SQL
│    └── PostgreSQL Database
│
└── Secret Manager
```

### Important next step

Before running `terraform apply`, run:

```powershell
terraform fmt
terraform validate
terraform plan
```

Also, **Cloud SQL and GKE can create real costs**, so carefully review `terraform plan` before applying.

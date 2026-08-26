# CI/CD Pipeline & Infrastructure Documentation

## Overview
This document outlines the Continuous Integration and Continuous Deployment (CI/CD) architecture for the E-Commerce Microservices Application. The infrastructure is entirely provisioned on **Oracle Cloud Infrastructure (OCI)** using **Terraform (Infrastructure as Code)**. The application deployment is automated using **GitHub Actions**, **Helm**, and **ArgoCD**, ensuring a robust, scalable, and fully automated GitOps workflow.

---

## 1. Architecture Components

### Infrastructure as Code (Terraform)
- **Engine**: Terraform v1.6.0
- **Cloud Provider**: Oracle Cloud Infrastructure (OCI)
- **State Management**: OCI Object Storage (S3-Compatible Backend) is used to store the `terraform.tfstate`. This ensures that GitHub Actions runners are idempotent and do not attempt to provision identical infrastructure twice upon re-runs.
- **Resources Provisioned**:
  - Virtual Cloud Network (VCN), Public and Private Subnets, and Network Security Groups (NSGs).
  - Oracle Kubernetes Engine (OKE) Cluster (v1.34.2) with ARM64 Node Pools.

### GitOps & Deployment Orchestration
- **ArgoCD**: Deployed into the OKE cluster to continuously monitor the `Terraform-code` repository. It automatically synchronizes the Kubernetes manifests and Helm charts to the cluster.
- **Helm**: Used to template and package the microservices application (`ecommerce-app`).
- **NGINX Ingress Controller**: Automatically provisioned via Helm within the CI/CD pipeline, configured dynamically with the OCI Load Balancer Subnet OCID to expose the application to the public internet.

---

## 2. CI/CD Workflows (GitHub Actions)

The CI/CD process is split into two primary workflow types: **Application CI (Microservices)** and **Infrastructure CD (Terraform)**.

### A. Application CI Workflows (`ci.yml`)
Each microservice repository (`auth-service`, `products-service`, `orders-service`, `payments-service`, `ecomm-ui`) contains a GitHub Actions workflow that handles building and publishing the container images.

**Key Features:**
1. **Multi-Architecture Builds**: The OKE cluster utilizes ARM64 instances. To ensure compatibility, the CI pipelines utilize Docker Buildx and QEMU to build images for both `linux/amd64` and `linux/arm64` architectures.
2. **Base Image Optimization**: Microservices utilize the `node:20-slim` Debian-based image to prevent QEMU emulation bugs (e.g., `Illegal instruction` core dumps) that occur with Alpine-based images during ARM64 cross-compilation.
3. **OCIR Integration**: Images are securely pushed to the Oracle Cloud Infrastructure Registry (OCIR). Authentication is handled via the native OCI CLI.

### B. Infrastructure CD Workflow (`deploy.yml`)
Located in the `Terraform-code` repository, the `deploy.yml` workflow orchestrates the end-to-end provisioning of the cloud environment and the deployment of the application.

**Workflow Stages:**
1. **Terraform Init & Apply**: 
   - Authenticates with OCI using Customer Secret Keys for the S3-compatible backend.
   - Applies the Terraform configuration to provision the VCN and OKE cluster.
2. **Dynamic Output Extraction**: 
   - Extracts the newly created `cluster_id` and `load_balancer_subnet_id` directly from the Terraform state to configure downstream components dynamically.
3. **Kubeconfig Generation**: 
   - Uses the OCI CLI to generate a secure `kubeconfig` to interact with the OKE cluster.
4. **NGINX Ingress Installation**: 
   - Installs the NGINX Ingress Controller via Helm.
   - Automatically injects the `service.beta.kubernetes.io/oci-load-balancer-subnet1` annotation using the extracted subnet OCID to ensure OCI provisions a Public IP.
5. **Cert-Manager & Automated TLS**:
   - Installs Jetstack's `cert-manager` via Helm.
   - Automatically deploys a Let's Encrypt `ClusterIssuer` configured with HTTP-01 challenges to dynamically provision and renew SSL certificates for secure HTTPS routing.
6. **Secret Management**: 
   - Creates a permanent Kubernetes Docker Registry Secret (`ocir-secret`) using the `OCI_AUTH_TOKEN` GitHub Secret, ensuring pods can continuously pull images without JWT expiration issues.
   - Provisions application secrets (JWT, Stripe, Admin credentials) securely into the cluster.
7. **ArgoCD Bootstrap**: 
   - Installs ArgoCD and applies the `ecommerce-app` Application manifest, transferring control of the application lifecycle to ArgoCD (GitOps).

---

## 3. Required GitHub Secrets & Variables

To securely execute the CI/CD pipelines, the following secrets must be configured in the GitHub Organization/Repository settings:

### Infrastructure & State Secrets
- `OCI_TENANCY_OCID`, `OCI_USER_OCID`, `OCI_FINGERPRINT`, `OCI_KEY_CONTENT`: OCI API Authentication.
- `OCI_COMPARTMENT_OCID`: Target compartment for resources.
- `OCI_S3_ACCESS_KEY` & `OCI_S3_SECRET_KEY`: Customer Secret Keys for Terraform S3 backend.
- `OCI_NAMESPACE`: OCI Object Storage Namespace.

### Application Secrets
- `OCI_AUTH_TOKEN`: Permanent Auth Token for OCIR docker login.
- `OCI_OCIR_USERNAME`: OCI Registry username.
- `JWT_SECRET`: Secret key for JWT signing.
- `STRIPE_SECRET_KEY`: Stripe payment gateway API key.
- `ADMIN_EMAIL` & `ADMIN_PASSWORD`: Default administrator credentials.
- `GH_PAT`: GitHub Personal Access Token for cross-repository access.

---

## 4. Resilience & Error Handling Implementations
- **Cascading Failure Prevention**: Application liveness and readiness probe timeouts were optimized to accommodate Server-Side Rendering (SSR) cold starts on ARM64 nodes, preventing premature pod termination.
- **Idempotency**: The implementation of the OCI S3-compatible backend ensures that re-running `deploy.yml` will safely update existing infrastructure rather than attempting to duplicate VCNs or clusters.

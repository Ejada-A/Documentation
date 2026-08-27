# Problems & Solutions Log

This document tracks the technical issues encountered during the deployment of the E-Commerce application to Oracle Cloud Infrastructure (OCI) and their corresponding solutions.

---

### 1. QEMU Core Dump / Illegal Instruction on ARM64 Nodes
**Problem:**  
The microservices were initially built using `node:20-alpine`. Because the target OKE (Oracle Kubernetes Engine) cluster uses ARM64 nodes, GitHub Actions (which runs on AMD64) used QEMU to cross-compile the images. Alpine's `musl` libc caused QEMU to crash during the build process with the error: `uncaught target signal 4 (Illegal instruction) - core dumped`.

**Solution:**  
Migrated all microservice Dockerfiles from `node:20-alpine` to `node:20-slim` (which is Debian-based and uses standard `glibc`). Also updated the `ci.yml` workflows to properly build multi-arch images (`linux/amd64,linux/arm64`) using Docker Buildx.

---

### 2. OCIR Authentication Expiration (`ImagePullBackOff`)
**Problem:**  
The infrastructure pipeline (`deploy.yml`) was generating the Kubernetes `ocir-secret` using a temporary token retrieved via `oci container-registry access-token get`. When this token naturally expired, Kubernetes failed to pull new images or restart pods, resulting in `ImagePullBackOff` errors across the cluster.

**Solution:**  
Generated a permanent OCI Auth Token. Updated `deploy.yml` to utilize GitHub Secrets (`OCI_AUTH_TOKEN` and `OCI_OCIR_USERNAME`) to create a permanent `docker-registry` secret, ensuring continuous and uninterrupted access to the Oracle Container Registry.

---

### 3. Application Cold-Start Probe Failures
**Problem:**  
The `ecomm-ui` (Next.js) pod consistently failed its Kubernetes Liveness and Readiness probes immediately upon deployment and entered a crash loop. This occurred because Server-Side Rendering (SSR) cold starts on ARM64 nodes take longer than the default Kubernetes probe timeouts.

**Solution:**  
Significantly increased the `initialDelaySeconds` and `timeoutSeconds` for both probes in the `ecomm-ui.yaml` Helm template. This provided the Next.js application with sufficient time to bootstrap and complete its initial rendering before Kubernetes marked it as unhealthy.

---

### 4. NGINX Ingress Public IP Stuck in `<pending>`
**Problem:**  
After deploying the NGINX Ingress Controller via Helm, its LoadBalancer service remained stuck in a `<pending>` state and never provisioned a public IP address.

**Solution:**  
OCI requires a specific subnet annotation to determine where to attach the Load Balancer. Updated `output.tf` to explicitly export `load_balancer_subnet_id`. Modified `deploy.yml` to dynamically extract this OCID and inject it into the Helm installation command via the `service.beta.kubernetes.io/oci-load-balancer-subnet1` annotation.

---

### 5. Terraform Local State Loss (Double Provisioning)
**Problem:**  
Because GitHub Actions runners are ephemeral, the local `terraform.tfstate` was destroyed after every pipeline run. Subsequent runs of `deploy.yml` attempted to provision the same VCNs, Subnets, and OKE clusters from scratch, causing pipeline failures due to resource conflicts.

**Solution:**  
Configured the OCI S3-Compatible remote backend in `providers.tf`. Updated `deploy.yml` to pass `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` (mapped to OCI Customer Secret Keys) during `terraform init` to persist state securely in a remote Oracle Object Storage bucket (`terraform-states`).

---

### 6. Incompatible SSL Certificates (AWS ACM vs OCI)
**Problem:**  
An SSL certificate was generated using AWS ACM (Amazon Certificate Manager) with the intention of attaching it to `shoy.publicvm.com`. However, AWS ACM certificates are locked exclusively to AWS infrastructure and their private keys cannot be exported, making them incompatible with an Oracle Cloud NGINX Ingress.

**Solution:**  
Discarded the AWS ACM certificate. Updated `deploy.yml` to automatically install Jetstack's `cert-manager`. Created a Let's Encrypt `ClusterIssuer` to automatically provision HTTP-01 challenged certificates and dynamically injected the `cert-manager.io/cluster-issuer: "letsencrypt-prod"` annotation into `ingress.yaml` and `values.yaml` to secure the domain.

---

### 7. Helm Execution Order Bug
**Problem:**  
The `deploy.yml` pipeline attempted to install NGINX and `cert-manager` via Helm *before* the OCI CLI had generated the `kubeconfig` authentication file, causing the Helm commands to fail.

**Solution:**  
Reordered the pipeline steps in `deploy.yml` so that `oci ce cluster create-kubeconfig` successfully runs and exports the artifact before any `kubectl` or `helm` installations are executed.

---

### 8. Ambiguous Image Pull Errors (MongoDB)
**Problem:**  
OKE worker nodes rejected the image `mongo:7.0` during pod scheduling due to a "short name mode enforcing" error, as the cluster could not determine the exact registry to pull from.

**Solution:**  
Updated `helm/ecommerce-app/values.yaml` to use the fully qualified image registry path: `docker.io/library/mongo:7.0`.

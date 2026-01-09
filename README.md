# Production-Grade WordPress Deployment on Kubernetes using Helm

## 📌 Overview

This project demonstrates how to deploy a **production-grade WordPress application on Kubernetes** using **Helm**, following real-world DevOps best practices.

The setup includes:

- **WordPress** as a scalable, stateless application
- **MySQL** as a stateful backend using a StatefulSet
- **Nginx (OpenResty)** as a reverse proxy in front of the application
- **Persistent storage** using Kubernetes Persistent Volumes
- **ReadWriteMany (RWX)** storage via NFS for shared WordPress content
- **Monitoring** using Prometheus and Grafana

The goal of this assignment is not just to run WordPress, but to **design and explain a production-ready architecture**, demonstrate Kubernetes concepts, and show practical DevOps troubleshooting skills.

---

## 🎯 Objectives Covered

- Deploy WordPress on Kubernetes in a production-oriented manner
- Use Helm charts for templated and repeatable deployments
- Separate stateless and stateful components correctly
- Implement persistent storage with appropriate access modes
- Design for scalability and high availability
- Add observability using Prometheus and Grafana
- Clearly document architectural decisions and known limitations

---

## 🧠 Design Philosophy

- **Security-first approach**  
  Sensitive information such as database credentials is injected via environment variables and Kubernetes resources, not baked into container images.

- **Separation of concerns**
  - **Nginx (OpenResty)** handles incoming traffic and acts as a reverse proxy
  - **WordPress** handles application logic and remains stateless
  - **MySQL** stores application data and runs as a stateful workload

- **Cloud-native principles**
  - Immutable container images
  - Declarative Kubernetes manifests
  - Externalized configuration and storage

- **Production realism**
  The focus is on correct architecture, storage design, and monitoring rather than shortcuts to simply make the application run locally.

---

## ✅ What This Project Demonstrates

This repository emphasizes:
- Correct Kubernetes workload selection (Deployment vs StatefulSet)
- Proper use of Persistent Volumes and Persistent Volume Claims
- RWX storage design for horizontally scaled applications
- Helm-based application lifecycle management
- Real-world troubleshooting of Kubernetes storage and local cluster limitations

The intent is to showcase **DevOps thinking and decision-making**, not just a working demo.

## 🏗️ Architecture Overview

The application follows a **layered, production-style architecture**, where each component has a clear responsibility and can be scaled or managed independently.

### High-Level Architecture
Client
  |
  v
NodePort Service (Nginx)
  |
  v
Nginx (OpenResty)
  |
  v
WordPress Service (ClusterIP)
  |
  v
WordPress Pods (Deployment)
  |
  +----------------------+
  |                      |
  v                      v
RWX Storage (NFS)   MySQL (StatefulSet, RWO)


---

## 🔧 Component Breakdown

### 1️⃣ Nginx (OpenResty)

- Acts as a **reverse proxy** in front of the WordPress application
- Prevents direct exposure of application pods
- Enables future support for:
  - Request logging
  - Rate limiting
  - Lua-based request metrics
- Deployed as a **Kubernetes Deployment**
- Exposed externally using a **NodePort Service**

**Why Nginx runs separately:**
- Reverse proxies should be independent from application containers
- Allows scaling, configuration changes, and observability without touching the app
- Follows real-world production patterns

---

### 2️⃣ WordPress

- Deployed as a **stateless application** using a Kubernetes Deployment
- Scaled horizontally using multiple replicas
- Stores:
  - Application code inside the container image
  - User-generated content (uploads, plugins, themes) on shared storage
- Connects to MySQL using internal Kubernetes DNS

**Why WordPress is stateless:**
- Pods can be created and destroyed at any time
- All state is externalized to:
  - MySQL (structured data)
  - Shared filesystem (unstructured data)

---

### 3️⃣ MySQL

- Deployed using a **StatefulSet**
- Uses a **Headless Service** for stable network identity
- Backed by a **ReadWriteOnce (RWO) Persistent Volume**
- Ensures:
  - Stable pod identity (`mysql-0`)
  - Stable storage attachment
  - Data consistency

**Why StatefulSet is required:**
- Databases require stable identity and exclusive write access
- Prevents multiple pods from writing to the same data volume

---

### 4️⃣ Persistent Storage Design

| Component   | Storage Type | Access Mode |
|------------|-------------|-------------|
| WordPress  | Shared NFS   | ReadWriteMany (RWX) |
| MySQL      | Block Volume | ReadWriteOnce (RWO) |

**Why RWX for WordPress:**
- Multiple WordPress replicas need access to the same:
  - Media uploads
  - Plugins
  - Themes
- Ensures consistency across all application pods

**Why RWO for MySQL:**
- Databases must allow **only one writer**
- Prevents data corruption and race conditions

---

### 5️⃣ ReadWriteMany (RWX) via NFS

- RWX storage is implemented using an **NFS-backed PersistentVolume**
- Enables horizontal scaling of WordPress pods
- In production, this would typically be replaced by:
  - Managed NFS
  - AWS EFS
  - Cloud-native file storage

---

## 🔁 Application Lifecycle

- Entire stack can be deployed using a **single Helm command**
- Application components can be updated independently
- Pods can be restarted without data loss due to externalized storage

This architecture closely mirrors how WordPress is deployed in **real production Kubernetes environments**.

## 📝 Notes to Deploy and Test

This section documents the steps to deploy the application using Helm and validate that all components work as expected in a Kubernetes environment.

==================================================
🚀 Deployment Notes
==================================================

Prerequisites:
- Kubernetes cluster (tested using kind)
- Helm v3+
- kubectl configured for the cluster
- Docker running (required for kind)
==================================================
STEP 0: Install Prerequisites
==================================================

This step installs all required tools before deploying the application.

--------------------------------------------------
Operating System
--------------------------------------------------
- Linux / WSL2 / macOS
- Docker must be available on the system

--------------------------------------------------
Install Docker
--------------------------------------------------

Verify Docker installation:

docker --version

If Docker is not installed, install Docker Desktop
and ensure the Docker daemon is running.

--------------------------------------------------
Install kubectl
--------------------------------------------------

Check if kubectl is installed:

kubectl version --client

Install kubectl (Linux):

curl -LO https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl
chmod +x kubectl
sudo mv kubectl /usr/local/bin/

--------------------------------------------------
Install Helm
--------------------------------------------------

Check Helm version:

helm version

Install Helm:

curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

--------------------------------------------------
Install kind (Kubernetes in Docker)
--------------------------------------------------

Check kind version:

kind version

Install kind:

curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.23.0/kind-linux-amd64
chmod +x kind
sudo mv kind /usr/local/bin/

--------------------------------------------------
Verify All Tools
--------------------------------------------------

docker --version
kubectl version --client
helm version
kind version

--------------------------------------------------
Optional (Recommended)
--------------------------------------------------

Install Git:

git --version

Install a browser to access WordPress and Grafana.

--------------------------------------------------
Prerequisites Installation Complete
--------------------------------------------------

--------------------------------------------------
Step 1: Create Kubernetes Cluster
--------------------------------------------------

kind create cluster --name wordpress-cluster
kubectl config use-context kind-wordpress-cluster

Verify cluster readiness:

kubectl get nodes

--------------------------------------------------
Step 2: Deploy the Application Using Helm
--------------------------------------------------

From the repository root directory:

helm install my-release helm/wordpress

This single command deploys:
- MySQL (StatefulSet)
- WordPress (Deployment)
- Nginx (OpenResty reverse proxy)
- Persistent Volumes and Persistent Volume Claims
- In-cluster NFS server (for RWX storage demonstration)

--------------------------------------------------
Step 3: Verify Deployment Status
--------------------------------------------------

Check pods:

kubectl get pods

Check services:

kubectl get svc

Check storage resources:

kubectl get pv
kubectl get pvc

Expected state:
- MySQL pod → Running
- Nginx pod → Running
- PVCs → Bound
- WordPress pods → Running (when RWX storage is available)

==================================================
🧪 Testing Notes
==================================================

--------------------------------------------------
Test 1: Application Reachability
--------------------------------------------------

Fetch the NodePort for Nginx:

kubectl get svc nginx

Access the application in a browser:

http://<NodeIP>:<NodePort>

Expected result:
- WordPress installation or home page is displayed

--------------------------------------------------
Test 2: Database Connectivity
--------------------------------------------------

- Complete the WordPress setup via the UI
- Successful setup confirms:
  - WordPress can connect to MySQL
  - Database credentials are correctly injected
  - MySQL StatefulSet is functioning properly

--------------------------------------------------
Test 3: RWX Storage Validation
--------------------------------------------------

RWX storage ensures all WordPress replicas share the same filesystem.

Steps:
1. Upload a media file via the WordPress admin dashboard
2. Scale WordPress replicas:

kubectl scale deployment wordpress --replicas=3

3. Refresh the application multiple times

Expected result:
- Uploaded media is accessible from all replicas
- Confirms shared RWX storage is working correctly

--------------------------------------------------
Test 4: Pod Restart & Persistence
--------------------------------------------------

Restart WordPress pods:

kubectl rollout restart deployment wordpress

Expected result:
- Pods restart successfully
- Media uploads and configuration persist after restart

--------------------------------------------------
Test 5: Monitoring Validation (Optional)
--------------------------------------------------

If monitoring is enabled using Prometheus and Grafana:
- Verify Prometheus targets are healthy
- Access Grafana dashboards
- Confirm visibility of:
  - Pod CPU and memory usage
  - Node metrics
  - Kubernetes object metrics

==================================================
🔄 Upgrade and Retest
==================================================

Apply configuration or image changes:

helm upgrade my-release helm/wordpress

Repeat the above tests to ensure:
- No data loss
- Stable application behavior

==================================================
⚠️ Local Environment Notes
==================================================

- RWX storage is demonstrated using an in-cluster NFS server
- Local Kubernetes environments (e.g., kind on WSL) may encounter NFS kernel limitations
- In production, this setup should use managed NFS solutions such as AWS EFS

These limitations do not affect the architectural correctness of the solution.

==================================================
🧹 Cleanup After Testing
==================================================

Remove the application:

helm uninstall my-release

Delete the Kubernetes cluster:

kind delete cluster --name wordpress-cluster

==================================================
✅ Summary
==================================================

Following these steps validates:
- Correct Helm-based deployment
- Proper storage configuration (RWX and RWO)
- Application scalability and persistence
- Production-style deployment and testing workflow


## 📊 Monitoring

Monitoring is deployed **separately from the application Helm chart**.

The WordPress Helm chart is responsible only for:
- Application workloads (WordPress, MySQL, Nginx)
- Networking and storage resources

Cluster-level observability is handled independently using
`kube-prometheus-stack`.

This separation follows production best practices where:
- Monitoring is shared across multiple applications
- Application lifecycle does not control monitoring lifecycle
- Monitoring remains available even if applications are redeployed or removed

--------------------------------------------------
Monitoring Deployment
--------------------------------------------------

Monitoring is installed using Helm:

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

helm install monitoring prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace

--------------------------------------------------
Monitoring Capabilities
--------------------------------------------------

The monitoring stack provides:
- Prometheus for metrics collection
- Grafana for visualization
- Alertmanager for alerting
- Node and Kubernetes object metrics

--------------------------------------------------
Metrics Observed
--------------------------------------------------

- Kubernetes node health
- Pod CPU and memory usage
- Deployment and StatefulSet status
- Nginx request metrics (when exposed)
- Cluster resource utilization

--------------------------------------------------
Accessing Grafana
--------------------------------------------------

kubectl get svc -n monitoring

Grafana dashboards validate that the application is:
- Running
- Scalable
- Observable
- Production-ready


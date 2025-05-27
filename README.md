# Bileasy_Assignment

## 📦 Setup Instructions

### 1. Prerequisites
- Minikube (v1.32+)
- kubectl
- Helm
- Docker
- `mkcert` (for local TLS certificates)

### 2. Start Minikube
```bash
minikube start --cpus=4 --memory=8192
```

### 3. Enable Ingress Addon
```bash
minikube addons enable ingress
```

### 4. Deploy Monitoring Stack
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

helm install prometheus prometheus-community/kube-prometheus-stack -n monitoring --create-namespace
```

### 5. Deploy Your Microservices
- Create a `kustomize` or `helm` setup with:
  - `auth-service`
  - `gateway`
  - `data-service`
  - MinIO


### 6. Apply RBAC and Kyverno Policies
- Define `ServiceAccount`, `Role`, and `RoleBinding` to restrict MinIO access to only `data-service`.
- Create Kyverno policy to deny any pod sending `Authorization` headers if it’s not `data-service`.

---

## 🗂 Architecture Diagram

```
                        +-------------------+
                        |   Grafana (UI)    |
                        +-------------------+
                                |
                                v
                         +--------------+
                         |  Prometheus  |
                         +--------------+
                                |
       +------------------------+-------------------------+
       |                        |                         |
+--------------+     +--------------------+     +-----------------------+
| Node Exporter|     | kube-state-metrics |     | Pod Metrics (cAdvisor)|
+--------------+     +--------------------+     +-----------------------+
                                                    |
                                                    v
                                   +--------------------------------+
                                   |  Microservices (Pods)          |
                                   |  - gateway                     |
                                   |  - auth-service                |
                                   |  - data-service                |
                                   |  - minio                       |
                                   +--------------------------------+
```

---

## 🎯 Design Decisions

- **Minikube** is chosen for simplicity and reproducibility.
- **Prometheus Stack** for monitoring due to native Kubernetes support.
- **Kyverno** for policy enforcement because it uses native Kubernetes resources (CRDs).
- **MinIO** as a test object store due to its AWS S3 compatibility.
- **RBAC & Secrets** ensure least privilege and secure access to MinIO.

---

## 🔐 Security Incident Simulation

### Incident:
The `auth-service` was observed sending `Authorization` headers downstream, potentially leaking credentials to other internal services or logs.

### Response:
- Created a Kyverno policy to block outbound HTTP requests from any pod (except `data-service`) containing the `Authorization` header.
- RBAC was applied to limit access to MinIO so that only `data-service` has read/write permissions.
- All sensitive data (e.g., MinIO credentials) is stored securely in Kubernetes `Secrets`.


---

## 📌 Assumptions & Known Issues

### Known Issues:
- Simulating stress loads may cause scheduling issues on low-resource machines.
- Minikube is not a production-ready environment; results may vary under actual cloud setups..

---

## 🧪 Failure Simulation (Part 5)

### Test Performed:
- Simulated partial failure by deleting the `auth-service` pod.
- Injected resource stress using an Alpine-based `stress` pod.
- Monitored system recovery via ReplicaSets and HPA.
- Observed system behavior using Grafana dashboard.

### Outcome:
- Deleted pods were recreated automatically by ReplicaSets.
- No unauthorized access or policy violations occurred during failure simulation.


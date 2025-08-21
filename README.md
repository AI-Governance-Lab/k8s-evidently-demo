# k8s-evidently-demo

📊 Evidently AI on Kubernetes — simple deployment demo.

This repository provides Kubernetes manifests to run [Evidently AI](https://github.com/evidentlyai/evidently) as a service inside your cluster (EKS, GKE, AKS, Minikube, etc.).
You can connect it to any AI Agent or ML model (RAG, Ollama, FastAPI, etc.) to monitor model performance, detect data drift, and generate reports.

---

## Index
- [Quickstart](#quickstart)
  - [0. Prerequisites](#0-prerequisites)
  - [1. Clone repo](#1-clone-repo)
  - [2. Deploy on Kubernetes](#2-deploy-on-kubernetes)
  - [3. Verify](#3-verify)
  - [4. Access Evidently (NodePort, default)](#4-access-evidently-nodeport-default)
  - [5. Optional: Access via Ingress](#5-optional-access-via-ingress)
  - [6. Optional: Port-forward (no cluster exposure)](#6-optional-port-forward-no-cluster-exposure)
- [Connecting to your AI Agent](#connect-ai-agent)
- [Architecture](#architecture)
- [AI Governance Considerations](#ai-governance)
- [Cleanup](#cleanup)
- [Next steps](#next-steps)

<a id="quickstart"></a>

## 🚀 Quickstart

### 0. Prerequisites
- A Kubernetes cluster and kubectl configured
- Optional: A namespace to isolate resources
- Optional: An Ingress Controller (e.g., NGINX) if you plan to use Ingress

Create a namespace (optional but recommended):
```bash
kubectl create namespace evidently
```

### 1. Clone repo
```bash
git clone https://github.com/AI-Governance-Lab/k8s-evidently-demo.git
cd k8s-evidently-demo
```

### 2. Deploy on Kubernetes
```bash
kubectl apply -n evidently -f k8s/deployment.yaml
kubectl apply -n evidently -f k8s/service.yaml
```

### 3. Verify
```bash
kubectl get pods -n evidently
kubectl get svc evidently-service -n evidently
```
Wait until the pod is Ready (1/1). If needed:
```bash
kubectl describe pods -l app=evidently -n evidently
kubectl logs deploy/evidently -n evidently
```

### 4. Access Evidently (NodePort, default)
- Get the node IP and open in a browser:
```bash
kubectl get svc evidently-service -n evidently
```
Then open: http://<node-ip>:30080

- Minikube shortcut:
```bash
minikube service evidently-service -n evidently --url
```

### 5. Optional: Access via Ingress
If you have an Ingress Controller and a manifest at k8s/ingress.yaml:
```bash
kubectl apply -n evidently -f k8s/ingress.yaml
kubectl get ingress -n evidently
```
Point your DNS (or hosts file) to the ingress host and open it in a browser.

### 6. Optional: Port-forward (no cluster exposure)
```bash
kubectl -n evidently port-forward deployment/evidently 8000:8000
```
Open: http://localhost:8000

<a id="connect-ai-agent"></a>

## 📡 Connecting to your AI Agent
Any AI agent or ML service can send data to Evidently:
- Log predictions/inputs in JSON/CSV
- Use Evidently’s Python client to generate reports
- Mount a shared volume or push results via API
- Check examples/sample_notebook.ipynb for a minimal workflow

<a id="architecture"></a>

## 🏛️ Architecture
```mermaid
flowchart TB
  %% Clients
  U[User / Stakeholders]
  DS[Data Scientist / Engineer]

  %% Cluster
  subgraph K8s[Kubernetes Cluster]
    A[Evidently Service\n(evidentlyai/evidently:latest)]
    SVC[Service\n(NodePort: 30080)]
    IG[Ingress Controller\n(Optional)]
    AG[AI Agent / Model API\n(FastAPI / Ollama / etc.)]
    PVC[(Persistent Volume / Storage)\nReports & Logs]
    EXP[Prometheus Exporter\n(Optional)]
  end

  %% Observability
  G[Grafana Dashboards\n(Optional)]
  PM[Prometheus\n(Optional)]

  %% Flows
  U -->|HTTP| IG
  IG -.->|Optional| SVC
  U -->|HTTP NodePort| SVC
  SVC --> A
  DS -->|Notebook / API| A
  AG -->|Predictions / Inputs| A
  A -->|Reports / Artifacts| PVC
  A -.->|/metrics| EXP
  EXP --> PM --> G

  %% Notes
  classDef opt fill:#eef6ff,stroke:#a3c4f3,color:#1c3d5a;
  class IG,EXP opt
```

<a id="ai-governance"></a>

## 🛡️ AI Governance Considerations
- Data governance: structure input/output logs, define schemas, and version datasets used for reports.
- PII and security: add PII redaction before logging; restrict access via network policies and RBAC.
- Drift and performance monitoring: track data drift, quality, and performance metrics; set thresholds and alerts.
- Observability: expose Prometheus metrics (optional) and build Grafana dashboards for continuous visibility.
- Lineage and reproducibility: persist reports and configs in storage; include model/version metadata.
- Compliance and audit: retain reports and event logs; ensure time-stamped artifacts and access logs.
- Change management: review/approve metric thresholds and dashboards; document changes in Git.
- Incident response: define alert routes and on-call; add runbooks for remediation.

<a id="cleanup"></a>

## 🧹 Cleanup
```bash
# If applied
kubectl delete -n evidently -f k8s/ingress.yaml || true

kubectl delete -n evidently -f k8s/service.yaml
kubectl delete -n evidently -f k8s/deployment.yaml

# Optional: remove namespace
kubectl delete namespace evidently
```

<a id="next-steps"></a>

## 🛠️ Next steps
- Add Prometheus exporter for continuous metrics
- Create Helm chart
- Build Grafana

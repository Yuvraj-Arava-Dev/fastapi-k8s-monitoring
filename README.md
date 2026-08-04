# 🚀 FastAPI Kubernetes Deployment with Helm-based Prometheus & Grafana

A production-ready blueprint for deploying a 3-endpoint **FastAPI** application to a local **`kind`** (Kubernetes in Docker) cluster. This repository now uses **Helm** to install **Prometheus** and **Grafana** instead of static monitoring YAML manifests.

## 🔍 Overview

This project demonstrates how to:
1. Containerize a FastAPI application with built-in Prometheus metric instrumentation.
2. Spin up a local Kubernetes cluster using **`kind`** with one control-plane and two worker nodes.
3. Deploy the FastAPI application with Kubernetes `Deployment` and `Service` manifests.
4. Install **Prometheus** and **Grafana** with **Helm** charts.
5. Configure Prometheus to scrape `/metrics` from the FastAPI app.
6. Create Grafana dashboards that visualize pod-level FastAPI metrics.

## 🏛️ Architecture & Flow

The app exposes metrics at `/metrics` and Prometheus scrapes the FastAPI service using Kubernetes DNS. Grafana connects to Prometheus and renders request rate, latency, and status code panels.

## 🛠️ Instructions to Execute the Project

```bash
# create a cluster with kind
kind create cluster --config kind-config.yaml --name dev-cluster

# build the docker image for the FastAPI app
docker build -t fastapi-app:latest ./app

# load the docker image into kind
kind load docker-image fastapi-app:latest --name dev-cluster

# deploy the FastAPI application
kubectl apply -f k8s-deployment/
```

### Install Prometheus with Helm

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

cat > prometheus-values.yaml <<'EOF'
server:
  service:
    type: NodePort
    nodePort: 30090

serverFiles:
  prometheus.yml:
    global:
      scrape_interval: 15s

    rule_files: []

    scrape_configs:
      - job_name: fastapi
        metrics_path: /metrics
        static_configs:
          - targets:
              - fastapi-service.monitoring-demo.svc.cluster.local:8000
EOF

helm install prometheus prometheus-community/prometheus -n monitoring-demo --create-namespace -f prometheus-values.yaml
```

### Install Grafana with Helm

```bash
cat > grafana-values.yaml <<'EOF'
service:
  type: NodePort
  nodePort: 30000
adminPassword: admin
persistence:
  enabled: false
EOF

helm install grafana grafana/grafana -n monitoring-demo -f grafana-values.yaml
```

### Validate deployment

```bash
kubectl get pods -n monitoring-demo
kubectl get svc -n monitoring-demo
```

### Access services

- FastAPI: `http://localhost:8000`
- Prometheus: `http://localhost:9090`
- Grafana: `http://localhost:3000`

> Note: `kind-config.yaml` maps host ports to the control-plane. If you use a different port mapping, adjust the Helm NodePort values or use `kubectl port-forward`.

## 📊 Configure pod metrics in Grafana

1. Open Grafana at `http://localhost:3000` and sign in with `admin` / `admin`.
2. Add Prometheus as a datasource if it is not already configured:
   - Name: `Prometheus`
   - Type: `Prometheus`
   - URL: `http://prometheus-server.monitoring-demo.svc.cluster.local`
   - Access: `Proxy`
3. Import the existing Kubernetes Pods dashboard:
   - Go to `Dashboards` -> `New` -> `Import`.
   - Enter dashboard ID `6417`.
   - Select the `Prometheus` datasource.
   - Click `Import`.
4. Open the imported dashboard and filter to the `monitoring-demo` namespace to view FastAPI pod metrics.
5. Generate traffic to the FastAPI app so the dashboard has fresh samples:
   ```bash
   curl http://localhost:8000/
   curl http://localhost:8000/metrics
   ```
6. Set the dashboard refresh interval to `5s` or `10s` for live pod metrics.

> If dashboard `6417` shows `No data`, verify that Prometheus is scraping Kubernetes pod/container metrics such as `container_cpu_usage_seconds_total`, `container_memory_working_set_bytes`, and kube-state-metrics series. The FastAPI `/metrics` scrape alone exposes application metrics, but imported Kubernetes pod dashboards also need cluster-level metrics.

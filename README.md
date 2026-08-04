# 🚀 FastAPI Kubernetes Deployment with Prometheus & Grafana Monitoring

A production-ready blueprint for deploying a 3-endpoint **FastAPI** application to a local **`kind`** (Kubernetes in Docker) cluster. This repository includes auto-provisioned **Prometheus** for metrics collection and alerting, along with **Grafana** for real-time dashboard visualization.

## 🔍 Overview

This project demonstrates how to:
1. Containerize a FastAPI application with built-in Prometheus metric instrumentation.
2. Spin up a multi-port local Kubernetes cluster using **`kind`**.
3. Deploy the application using Kubernetes `Deployment` and `Service` (NodePort) manifests.
4. Set up **Prometheus** to scrape application metrics (`/metrics`) automatically using internal K8s DNS.
5. Configure **Prometheus Alert Rules** for high latency and 5xx error rates.
6. Auto-provision **Grafana** with a pre-configured dashboard displaying Requests Per Second (RPS), Latency (p95), and HTTP Status Codes.

## 🏛️ Architecture & Flow

<img width="472" height="263" alt="image" src="https://github.com/user-attachments/assets/b9ed70f8-41be-4f30-90ca-cb94658acbd9" />

# express-microservice-gitops


# 🚀 Cloud-Native GitOps Continuous Delivery Pipeline

[![GitOps](https://img.shields.io/badge/GitOps-ArgoCD-blue?style=for-the-badge&logo=argo)](https://argoproj.github.io/cd/)
[![Kubernetes](https://img.shields.io/badge/Orchestration-Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)](https://kubernetes.io/)
[![GitHub Actions](https://img.shields.io/badge/CI%2FCD-GitHub_Actions-2088FF?style=for-the-badge&logo=githubactions&logoColor=white)](https://github.com/features/actions)
[![Docker](https://img.shields.io/badge/Containerization-Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)](https://www.docker.com/)
[![Node.js](https://img.shields.io/badge/Backend-Node.js-339933?style=for-the-badge&logo=nodedotjs&logoColor=white)](https://nodejs.org/)

An automated, end-to-end GitOps Continuous Delivery pipeline for containerized microservices. This repository demonstrates automated testing, multi-stage container builds, automated manifest updates, and declarative pull-based continuous deployment into a local Kubernetes cluster using ArgoCD.

---

## 🏗️ Architecture & GitOps Flow

```text
+----------------------+         +---------------------------+         +-----------------------+
|  Developer Push      |         |  Application CI           |         |  Container Registry   |
|  express-app repo    | ------> |  GitHub Actions           | ------> |  Docker Hub           |
|  (App Code + Test)   |         |  - Unit Tests (Jest)      |         |  - Immutable Tags     |
+----------------------+         |  - Docker Multi-Stage     |         |    (Git SHA)          |
                                 |  - Updates Config Repo    |         +-----------------------+
                                 +---------------------------+                     |
                                               |                                   |
                                               v                                   |
+----------------------+         +---------------------------+                     |
|  Target Deployment   |         |  GitOps Sync Engine       |                     |
|  Kubernetes Cluster  | <------ |  ArgoCD Controller        | <-------------------+
|  (Pods, ReplicaSet,  |  Pull   |  - Polls Config Repo      |    Pull Image
|   Service)           |         |  - Self-Healing & Prune   |
+----------------------+         +---------------------------+


Key Engineering Decisions
  Decoupled Repositories: Separated source code (express-microservice-app) from infrastructure declarations (express-microservice-gitops) to enforce security isolation and least-privilege deployment permissions.

  Immutable Commit SHA Tagging: Replaced non-deterministic latest tags with Git commit SHAs (${{ github.sha }}) for precise auditability, trace tracking, and instant rollbacks.

  Declarative Synchronization: Configured ArgoCD with selfHeal and prune enabled, preventing cluster drift and maintaining 100% parity with Git as the single source of truth.

🛠️ Tech Stack & Key Tools
Application: Node.js (Express), Jest, Supertest

Containerization: Docker (Multi-stage builds, rootless execution)

CI Engine: GitHub Actions (Automated testing, build caching, cross-repo manifest updates)

CD Engine: ArgoCD (GitOps controller)

Orchestration: Kubernetes (Kind / Minikube)

📂 Repository Structure

gitops-project/
├── express-microservice-app/              # Application Source Code
│   ├── .github/workflows/
│   │   └── ci.yml                         # Automated CI & Manifest Updater
│   ├── src/                               # App Source & Health Checks
│   ├── tests/                             # Unit Test Suite
│   └── Dockerfile                         # Production-grade Multi-Stage Dockerfile
│
└── express-microservice-gitops/           # Infrastructure & Desired State
    ├── argocd-app.yaml                    # Declarative ArgoCD Application Manifest
    └── k8s/
        ├── deployment.yaml                # Deployment Specs (Probes, Resource Limits)
        └── service.yaml                   # ClusterIP Service Mapping

🚀 Quickstart & Setup Guide
Prerequisites
  Docker Desktop

  Kind or Minikube

  kubectl

1. Cluster Setup & ArgoCD Installation
# Create local Kubernetes cluster
kind create cluster --name gitops-cluster

# Create dedicated namespace and install ArgoCD
kubectl create namespace argocd
kubectl apply -n argocd --server-side -f [https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml](https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml)

# Verify all pods are running
kubectl get pods -n argocd

2. Deploy Declarative GitOps Application
# Apply ArgoCD Application Manifest
kubectl apply -f argocd-app.yaml

# Monitor application deployment state
kubectl get pods -n default --watch

3. Expose Service & Test API

# Forward application service port locally
kubectl port-forward svc/express-app-service 3000:80

# Query API in a separate terminal
curl http://localhost:3000

📸 Pipeline Verification & ScreenshotsStageProof of ExecutionAutomated CI BuildArgoCD TopologyCluster Pod Status


🔒 Security & Reliability Features
Non-Root Containers: App runs under the unprivileged node user in Docker to mitigate container breakout risks.

Production Resource Guards: Configured CPU/Memory requests and limits in deployment.yaml to eliminate noisy-neighbor interference.

Probes: Embedded /healthz liveness and readiness endpoints for Kubernetes traffic shaping and automated pod restarts upon failure.


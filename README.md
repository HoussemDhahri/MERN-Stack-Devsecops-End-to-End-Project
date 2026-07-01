<div align="center">

# 🧬 MERN Stack — DevOps End-to-End Project

<img src="https://img.shields.io/badge/DevOps-End--to--End-blueviolet?style=for-the-badge&logo=devops&logoColor=white"/>
<img src="https://img.shields.io/badge/Jenkins-Pipeline-D24939?style=for-the-badge&logo=jenkins&logoColor=white"/>
<img src="https://img.shields.io/badge/Docker-Containerized-2496ED?style=for-the-badge&logo=docker&logoColor=white"/>
<img src="https://img.shields.io/badge/Kubernetes-GitOps-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white"/>
<img src="https://img.shields.io/badge/ArgoCD-App--of--Apps-EF7B4D?style=for-the-badge&logo=argo&logoColor=white"/>
<img src="https://img.shields.io/badge/MongoDB-Database-47A248?style=for-the-badge&logo=mongodb&logoColor=white"/>
<img src="https://img.shields.io/badge/React-Frontend-61DAFB?style=for-the-badge&logo=react&logoColor=black"/>
<img src="https://img.shields.io/badge/Node.js-Backend-339933?style=for-the-badge&logo=nodedotjs&logoColor=white"/>
<img src="https://img.shields.io/badge/Prometheus-Monitoring-E6522C?style=for-the-badge&logo=prometheus&logoColor=white"/>
<img src="https://img.shields.io/badge/Grafana-Dashboards-F46800?style=for-the-badge&logo=grafana&logoColor=white"/>
<img src="https://img.shields.io/badge/Kustomize-Overlays-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white"/>

<br/>
<br/>

> **A production-grade DevOps pipeline** that automates the full software delivery lifecycle of a MERN (MongoDB, Express, React, Node.js) application — from source code to a fully monitored, autoscaled Kubernetes deployment — using GitOps with an ArgoCD **App-of-Apps** pattern.

</div>

---

## 📋 Table of Contents

- [🎯 Overview](#-overview)
- [🏗️ Architecture](#️-architecture)
- [📁 Project Structure](#-project-structure)
- [🔄 CI/CD Pipeline](#-cicd-pipeline)
- [☸️ Kubernetes & GitOps](#️-kubernetes--gitops)
- [📈 Monitoring Stack](#-monitoring-stack)
- [⚙️ Prerequisites](#️-prerequisites)
- [🚀 Getting Started](#-getting-started)
- [🌍 Environments](#-environments)
- [📊 Autoscaling (HPA)](#-autoscaling-hpa)

---

## 🎯 Overview

This project implements a **complete DevOps pipeline** for a **MERN Stack** application (MongoDB, Express/Node.js backend, React frontend). It demonstrates industry best practices for:

| Pillar | Implementation |
|--------|---------------|
| 🔄 **Continuous Integration** | Jenkins pipelines (separate `Jenkinsfile` for backend & frontend) triggered on every push |
| 📦 **Containerization** | Independent Dockerfiles per service, orchestrated locally via `docker-compose.yaml` |
| 🚢 **Continuous Delivery** | GitOps with **ArgoCD App-of-Apps** — Staging & Prod applications managed declaratively |
| ☸️ **Orchestration** | Kubernetes with Kustomize (`base` + environment `overlays`) |
| 🗄️ **Database** | MongoDB deployed as a `StatefulSet`, managed via `mongo-express` UI |
| 📈 **Autoscaling** | Horizontal Pod Autoscaler (HPA) for both backend & frontend, patched per environment |
| 📊 **Monitoring** | Prometheus + Grafana + Alertmanager (Telegram alerts), installed via Helm in a dedicated `monitoring` namespace |

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          DEVELOPER WORKFLOW                              │
│                                                                          │
│   git push ──► GitHub ──► Webhook ──► Jenkins (backend / frontend)      │
└──────────────────────────────────┬──────────────────────────────────────┘
                                   │
                    ┌──────────────▼──────────────┐
                    │       JENKINS CI/CD          │
                    │                              │
                    │  ✅ Checkout & Build          │
                    │  ✅ Install Dependencies      │
                    │  🐳 Docker Build (per service)│
                    │  📤 Push to Registry          │
                    │  🔄 Update Kustomize Image    │
                    │  🔄 Git Push (GitOps repo)    │
                    └──────┬────────────┬──────────┘
                           │            │
               ┌───────────▼──┐    ┌────▼────────────┐
               │   Registry   │    │   GitHub Repo    │
               │  (Docker)    │    │ (GitOps Source)  │
               └──────────────┘    └────────┬─────────┘
                                            │
                               ┌────────────▼─────────────┐
                               │      ArgoCD (Helm)        │
                               │   App-of-Apps Pattern     │
                               └──┬──────────┬────────┬───┘
                                  │          │        │
                      ┌───────────▼──┐  ┌────▼──────┐ │
                      │  K8s STAGING │  │ K8s PROD  │ │
                      │  namespace   │  │ namespace │ │
                      │              │  │           │ │
                      │  Frontend    │  │ Frontend  │ │
                      │  Backend     │  │ Backend   │ │
                      │  MongoDB     │  │ MongoDB   │ │
                      │  Mongo-Expr  │  │ Mongo-Expr│ │
                      │  HPA         │  │ HPA       │ │
                      └──────────────┘  └───────────┘ │
                                                       │
                                          ┌────────────▼──────────────┐
                                          │   monitoring namespace     │
                                          │      (Helm install)        │
                                          │                            │
                                          │  📈 Prometheus              │
                                          │  📊 Grafana                 │
                                          │  🔔 Alertmanager → Telegram │
                                          └────────────────────────────┘
```

> ℹ️ **ArgoCD** and the **kube-prometheus-stack** are both installed via **Helm** into their own dedicated namespaces (`argocd` and `monitoring`), while the application workloads themselves are deployed declaratively through the **App-of-Apps** ArgoCD pattern.

---

## 📁 Project Structure

```
MERN-Stack-Devops-End-to-End-Project/
│
├── Application-Code/
│   ├── backend/                        # Node.js / Express API source
│   ├── frontend/                       # React application source
│   └── docker-compose.yaml             # Local multi-container dev environment
│
├── Jenkins/
│   ├── Jenkinsfile-backend              # CI/CD pipeline for the backend service
│   └── Jenkinsfile-frontend             # CI/CD pipeline for the frontend service
│
└── Kubernetes-Manifests-file/
    │
    ├── argocd/
    │   ├── applications/
    │   │   ├── monitoring.yaml         # ArgoCD Application → monitoring stack
    │   │   ├── prod.yaml               # ArgoCD Application → production overlay
    │   │   └── staging.yaml            # ArgoCD Application → staging overlay
    │   └── app-of-apps.yaml            # Root ArgoCD Application (App-of-Apps)
    │
    ├── base/
    │   ├── backend/
    │   │   ├── deployment.yaml
    │   │   ├── hpa.yaml
    │   │   ├── kustomization.yaml
    │   │   └── service.yaml
    │   │
    │   ├── database/
    │   │   ├── configmap.yaml
    │   │   ├── kustomization.yaml
    │   │   ├── secret.yaml
    │   │   ├── service.yaml
    │   │   └── statefulset.yaml
    │   │
    │   ├── frontend/
    │   │   ├── configmap.yaml
    │   │   ├── deployment.yaml
    │   │   ├── hpa.yaml
    │   │   ├── ingress.yaml
    │   │   ├── kustomization.yaml
    │   │   └── service.yaml
    │   │
    │   └── mongo-express/
    │       ├── deployment.yaml
    │       ├── kustomization.yaml
    │       ├── secret.yaml
    │       └── service.yaml
    │
    ├── monitoring/                      # See monitoring-readme.md for details
    │   ├── alertmanager/
    │   ├── grafana/
    │   ├── helm/
    │   ├── prometheus/
    │   └── kustomization.yaml
    │
    ├── namespaces/
    │   ├── monitoring.yaml
    │   ├── prod.yaml
    │   └── staging.yaml
    │
    └── overlays/
        ├── prod/
        │   ├── configmap-patch.yaml
        │   ├── hpa-backend-patch.yaml
        │   ├── hpa-frontend-patch.yaml
        │   ├── ingress-patch.yaml
        │   └── kustomization.yaml
        │
        └── staging/
            ├── configmap-patch.yaml
            ├── hpa-backend-patch.yaml
            ├── hpa-frontend-patch.yaml
            ├── ingress-patch.yaml
            └── kustomization.yaml
```

---

## 🔄 CI/CD Pipeline

Two independent Jenkins pipelines handle each service separately, triggered on push to their respective paths:

```
🧹 Clean Workspace
    │
    ▼
📥 Checkout (GitHub)
    │
    ▼
🏷️  Set Image Tag ──────────── git commit SHA / build number
    │
    ▼
⚙️  Install Dependencies (npm ci)
    │
    ▼
🧪 Lint + Unit Tests
    │
    ▼
🐳 Docker Build ──────────────── backend or frontend image
    │
    ▼
📤 Push to Registry
    │
    ▼
🔄 Update Kustomize Image Tag ── kustomize edit set image
    │
    ▼
🔄 Git Push (GitOps repo) ────── overlays/staging (or prod)
    │
    ▼
♻️  ArgoCD Auto-Sync
```

### Pipelines

| Pipeline | Trigger Path | Manifest Updated |
|----------|--------------|-------------------|
| `Jenkinsfile-backend` | `Application-Code/backend/**` | `base/backend` image tag |
| `Jenkinsfile-frontend` | `Application-Code/frontend/**` | `base/frontend` image tag |

### Local Development

```bash
cd Application-Code
docker-compose up -d
```

This spins up the **frontend**, **backend**, and **MongoDB** locally for fast iteration before pushing to the pipeline.

---

## ☸️ Kubernetes & GitOps

This project follows the **App-of-Apps** GitOps pattern with ArgoCD.

### How It Works

1. `app-of-apps.yaml` is the single root Application applied to the cluster.
2. It manages three child Applications defined under `argocd/applications/`:
   - `staging.yaml` → syncs `overlays/staging`
   - `prod.yaml` → syncs `overlays/prod`
   - `monitoring.yaml` → syncs the `monitoring/` Helm chart values
3. Jenkins updates the image tag inside the relevant overlay's `kustomization.yaml` and pushes to `main`.
4. ArgoCD detects the diff and automatically syncs the corresponding namespace.

### Namespaces

| Namespace | Manifest | Purpose |
|-----------|----------|---------|
| `staging` | `namespaces/staging.yaml` | Staging environment workloads |
| `prod` | `namespaces/prod.yaml` | Production environment workloads |
| `monitoring` | `namespaces/monitoring.yaml` | Prometheus / Grafana / Alertmanager (Helm) |
| `argocd` | *(installed via Helm)* | ArgoCD controller & UI |

### Application Components (per environment)

| Component | Manifest Source | Type |
|-----------|------------------|------|
| **Frontend** | `base/frontend` | Deployment + Service + Ingress + HPA |
| **Backend** | `base/backend` | Deployment + Service + HPA |
| **Database** | `base/database` | StatefulSet + Service + Secret |
| **Mongo Express** | `base/mongo-express` | Deployment + Service (DB admin UI) |

### Kustomize Overlay Structure

```yaml
# overlays/staging/kustomization.yaml
resources:
  - ../../base/backend
  - ../../base/frontend
  - ../../base/database
  - ../../base/mongo-express

patches:
  - configmap-patch.yaml
  - hpa-backend-patch.yaml
  - hpa-frontend-patch.yaml
  - ingress-patch.yaml
```

---

## 📈 Monitoring Stack

The full observability stack (Prometheus, Grafana, Alertmanager) is deployed via **Helm** into the `monitoring` namespace, and wired into ArgoCD as its own Application.

👉 Full details, dashboards, alert rules, and Telegram integration are documented separately in **[monitoring-readme.md](./monitoring-readme.md)**.

---

## ⚙️ Prerequisites

| Tool | Purpose | Version |
|------|---------|---------|
| **Jenkins** | CI/CD orchestration | LTS |
| **Node.js** | Build environment (backend & frontend) | 18+ |
| **Docker** | Container runtime | 24+ |
| **Kustomize** | K8s manifest patching | v5+ |
| **ArgoCD** | GitOps controller (installed via Helm) | v2.x |
| **Kubernetes** | Container orchestration | v1.28+ |
| **Helm** | Kubernetes package manager | v3+ |
| **Prometheus + Grafana** | Metrics & dashboards (kube-prometheus-stack) | Latest |
| **Metrics Server** | Required for HPA to function | Latest |

### Jenkins Credentials Required

| Credential ID | Type | Usage |
|--------------|------|-------|
| `github-token` | Username/Password | GitHub checkout & GitOps push |
| `dockerhub-creds` | Username/Password | Registry image push |

---

## 🚀 Getting Started

### 1. Clone the Repository

```bash
git clone https://github.com/<your-username>/MERN-Stack-Devops-End-to-End-Project.git
cd MERN-Stack-Devops-End-to-End-Project
```

### 2. Install ArgoCD (Helm)

```bash
helm repo add argo https://argoproj.github.io/argo-helm
kubectl create namespace argocd
helm install argocd argo/argo-cd -n argocd
```

### 3. Create Namespaces

```bash
kubectl apply -f Kubernetes-Manifests-file/namespaces/
```

### 4. Bootstrap with App-of-Apps

```bash
kubectl apply -f Kubernetes-Manifests-file/argocd/app-of-apps.yaml
```

This single command bootstraps **staging**, **prod**, and **monitoring** through ArgoCD automatically.

### 5. Configure Jenkins

- Create two **Pipeline** jobs, one pointing to `Jenkins/Jenkinsfile-backend` and one to `Jenkins/Jenkinsfile-frontend`
- Enable **GitHub webhook trigger** on both
- Add the required credentials (`github-token`, `dockerhub-creds`)

### 6. Trigger the Pipelines

```bash
git push origin main
# Jenkins webhook fires → backend and/or frontend pipeline starts automatically
```

---

## 🌍 Environments

### Staging
- Namespace: `staging`
- Synced by ArgoCD from: `overlays/staging/`
- Auto-updated on every successful pipeline run

### Production
- Namespace: `prod`
- Synced by ArgoCD from: `overlays/prod/`
- Config, ingress, and HPA thresholds patched independently from staging

---

## 📊 Autoscaling (HPA)

Both **backend** and **frontend** deployments ship with a base `hpa.yaml`, further tuned per environment:

| Overlay File | Purpose |
|--------------|---------|
| `hpa-backend-patch.yaml` | Adjusts min/max replicas & CPU/memory thresholds for the backend per environment |
| `hpa-frontend-patch.yaml` | Adjusts min/max replicas & CPU/memory thresholds for the frontend per environment |

> Production overlays typically define higher minimum replica counts and more conservative thresholds compared to staging.

---

<div align="center">

**Built with ❤️ — MERN Stack DevOps End-to-End Project**

<img src="https://img.shields.io/badge/GitOps-ArgoCD-orange?style=flat-square"/>
<img src="https://img.shields.io/badge/Pipeline-Jenkins-D24939?style=flat-square"/>
<img src="https://img.shields.io/badge/Monitoring-Prometheus%20%2B%20Grafana-F46800?style=flat-square"/>
<img src="https://img.shields.io/badge/Alerts-Telegram-26A5E4?style=flat-square"/>
<img src="https://img.shields.io/badge/License-MIT-green?style=flat-square"/>

</div>
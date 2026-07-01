<div align="center">

# 📈 Monitoring Stack — MERN Stack DevOps Project

<img src="https://img.shields.io/badge/Prometheus-Metrics-E6522C?style=for-the-badge&logo=prometheus&logoColor=white"/>
<img src="https://img.shields.io/badge/Grafana-Dashboards-F46800?style=for-the-badge&logo=grafana&logoColor=white"/>
<img src="https://img.shields.io/badge/Alertmanager-Alerts-E6522C?style=for-the-badge&logo=prometheus&logoColor=white"/>
<img src="https://img.shields.io/badge/Telegram-Notifications-26A5E4?style=for-the-badge&logo=telegram&logoColor=white"/>
<img src="https://img.shields.io/badge/Helm-Package%20Manager-0F1689?style=for-the-badge&logo=helm&logoColor=white"/>
<img src="https://img.shields.io/badge/ArgoCD-GitOps-EF7B4D?style=for-the-badge&logo=argo&logoColor=white"/>

<br/>
<br/>

> Full observability stack for the MERN application — **metrics**, **dashboards**, and **real-time alerting** — deployed via **Helm** into a dedicated `monitoring` namespace and managed declaratively through **ArgoCD**.

</div>

---

## 📋 Table of Contents

- [🎯 Overview](#-overview)
- [🏗️ Architecture](#️-architecture)
- [📁 Monitoring Folder Structure](#-monitoring-folder-structure)
- [☸️ Installation (Helm + ArgoCD)](#️-installation-helm--argocd)
- [📈 Prometheus](#-prometheus)
- [📊 Grafana](#-grafana)
- [🔔 Alertmanager & Telegram](#-alertmanager--telegram)
- [🗂️ ServiceMonitors](#️-servicemonitors)
- [🚨 Alert Rules](#-alert-rules)
- [🔐 Secrets](#-secrets)
- [🚀 Quick Start](#-quick-start)

---

## 🎯 Overview

The monitoring stack observes the **backend**, **frontend**, and **database** components of the MERN application across both `staging` and `prod` namespaces. It is composed of:

| Component | Role |
|-----------|------|
| 📈 **Prometheus** | Scrapes metrics from backend & database via `ServiceMonitor` resources, evaluates alert rules |
| 📊 **Grafana** | Provisioned dashboards for backend, frontend, and database observability |
| 🔔 **Alertmanager** | Routes firing alerts to a **Telegram bot** in real time |
| ⚓ **Helm** | Installs the `kube-prometheus-stack` release into the `monitoring` namespace |
| 🔄 **ArgoCD** | Tracks `monitoring/` and its Helm values as a dedicated Application (`monitoring.yaml`) |

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     staging / prod namespaces                    │
│                                                                   │
│   Backend Pods ──────► /metrics endpoint                        │
│   Database Pods ─────► /metrics endpoint (exporter)             │
└───────────────────┬───────────────────────┬─────────────────────┘
                    │  scrape (ServiceMonitor)                     
                    ▼                       ▼
┌─────────────────────────────────────────────────────────────────┐
│                     monitoring namespace (Helm)                  │
│                                                                   │
│   ┌──────────────┐      ┌──────────────┐      ┌───────────────┐ │
│   │  Prometheus   │────► │ Alertmanager │────► │   Telegram    │ │
│   │ (TSDB+rules)  │      │  (routing)   │      │     Bot       │ │
│   └──────┬───────┘      └──────────────┘      └───────────────┘ │
│          │                                                       │
│          ▼                                                       │
│   ┌──────────────┐                                               │
│   │   Grafana     │  ← Dashboards: backend / frontend / database │
│   └──────────────┘                                               │
└─────────────────────────────────────────────────────────────────┘
                    ▲
                    │ syncs Helm values + K8s objects
        ┌───────────┴────────────┐
        │  ArgoCD Application     │
        │  monitoring.yaml         │
        └─────────────────────────┘
```

---

## 📁 Monitoring Folder Structure

```
Kubernetes-Manifests-file/monitoring/
│
├── alertmanager/
│   ├── files/
│   │   ├── alertmanager.yaml         # Routing config (receivers, routes)
│   │   └── telegram.tmpl             # Custom Telegram message template
│   ├── kustomization.yaml
│   └── telegram-bot-secret.yaml      # Telegram bot token & chat ID (Secret)
│
├── grafana/
│   ├── dashboards/
│   │   ├── backend-dashboard.yaml    # Backend service dashboard (ConfigMap)
│   │   ├── database-dashboard.yaml   # MongoDB dashboard (ConfigMap)
│   │   └── frontend-dashboard.yaml   # Frontend dashboard (ConfigMap)
│   ├── grafana-admin-secret.yaml     # Grafana admin credentials
│   └── kustomization.yaml
│
├── helm/
│   └── values.yaml                   # kube-prometheus-stack Helm values
│
├── prometheus/
│   ├── rules/
│   │   ├── backend.yaml              # Backend alerting rules
│   │   ├── database.yaml             # Database alerting rules
│   │   └── infra.yaml                # Node/cluster infra alerting rules
│   ├── servicemonitors/
│   │   ├── backend-sm.yaml           # Scrape config for backend
│   │   └── database-sm.yaml          # Scrape config for database
│   └── kustomization.yaml
│
└── kustomization.yaml                 # Aggregates prometheus + grafana + alertmanager
```

---

## ☸️ Installation (Helm + ArgoCD)

Both **ArgoCD** and the **monitoring stack** are installed via **Helm**, each in its own namespace:

| Release | Namespace | Installed Via |
|---------|-----------|----------------|
| `argocd` | `argocd` | Helm (`argo/argo-cd`) |
| `kube-prometheus-stack` | `monitoring` | Helm, values sourced from `monitoring/helm/values.yaml`, tracked by ArgoCD |

### ArgoCD Application for Monitoring

```yaml
# argocd/applications/monitoring.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: monitoring
  namespace: argocd
spec:
  source:
    chart: kube-prometheus-stack
    repoURL: https://prometheus-community.github.io/helm-charts
    targetRevision: <chart-version>
    helm:
      valueFiles:
        - $values/Kubernetes-Manifests-file/monitoring/helm/values.yaml
  destination:
    namespace: monitoring
    server: https://kubernetes.default.svc
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

> This Application is one of the three children registered under `app-of-apps.yaml`, alongside `staging.yaml` and `prod.yaml`.

---

## 📈 Prometheus

Prometheus is configured to scrape both the **backend** and **database** services using `ServiceMonitor` custom resources, and evaluates custom `PrometheusRule` alert definitions.

### ServiceMonitors

| File | Target | Description |
|------|--------|-------------|
| `backend-sm.yaml` | Backend Service | Scrapes `/metrics` on the Node.js/Express API |
| `database-sm.yaml` | Database Service | Scrapes MongoDB exporter metrics |

### Rules

| File | Scope | Examples of Alerts |
|------|-------|---------------------|
| `backend.yaml` | Backend API | High error rate, high latency, pod restarts |
| `database.yaml` | MongoDB | Connection saturation, replication lag, high memory |
| `infra.yaml` | Cluster/Node | Node not ready, disk pressure, high CPU/memory usage |

---

## 📊 Grafana

Grafana dashboards are provisioned automatically as Kubernetes `ConfigMaps`, picked up by the Grafana sidecar dashboard loader.

| Dashboard | File | Content |
|-----------|------|---------|
| **Backend** | `backend-dashboard.yaml` | Request rate, latency (p95/p99), error rate, pod resource usage |
| **Frontend** | `frontend-dashboard.yaml` | Traffic, response codes, ingress metrics |
| **Database** | `database-dashboard.yaml` | Connections, operations/sec, replication status, memory/disk usage |

### Access

```bash
kubectl -n monitoring port-forward svc/kube-prometheus-stack-grafana 3000:80
```

Then open [http://localhost:3000](http://localhost:3000) and log in using the credentials from `grafana-admin-secret.yaml`.

---

## 🔔 Alertmanager & Telegram

Alertmanager routes firing Prometheus alerts to a **Telegram bot** for real-time notifications.

### Configuration Files

| File | Purpose |
|------|---------|
| `alertmanager.yaml` | Defines routes, receivers, grouping, and repeat intervals |
| `telegram.tmpl` | Custom message template (severity, alert name, description, environment) |
| `telegram-bot-secret.yaml` | Stores the Telegram **bot token** and **chat ID** as a Kubernetes Secret |

### Example Route

```yaml
route:
  receiver: telegram-notifications
  group_by: ['alertname', 'namespace', 'severity']
  routes:
    - match:
        severity: critical
      receiver: telegram-notifications
      repeat_interval: 15m

receivers:
  - name: telegram-notifications
    telegram_configs:
      - bot_token_file: /etc/alertmanager/secrets/telegram-bot-secret/bot-token
        chat_id: <chat-id>
        parse_mode: 'HTML'
        message: '{{ template "telegram.tmpl" . }}'
```

> Sensitive values (bot token, chat ID) are never committed in plain text — they are stored in `telegram-bot-secret.yaml` as a Kubernetes `Secret`.

---

## 🗂️ ServiceMonitors

`ServiceMonitor` resources tell Prometheus **what to scrape** and **how often**, matching services by label selector:

```yaml
# prometheus/servicemonitors/backend-sm.yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: backend-servicemonitor
  namespace: monitoring
spec:
  selector:
    matchLabels:
      app: backend
  endpoints:
    - port: http
      path: /metrics
      interval: 15s
  namespaceSelector:
    matchNames:
      - staging
      - prod
```

---

## 🚨 Alert Rules

Alert rules are grouped by domain under `prometheus/rules/` and loaded as `PrometheusRule` CRDs.

| Category | Example Alerts |
|----------|-----------------|
| **Backend** | `BackendHighErrorRate`, `BackendHighLatency`, `BackendPodCrashLooping` |
| **Database** | `MongoDBDown`, `MongoDBHighConnections`, `MongoDBReplicationLag` |
| **Infra** | `NodeNotReady`, `NodeHighCPU`, `NodeDiskPressure` |

Each rule fires into Alertmanager, which forwards the notification to the configured **Telegram** channel.

---

## 🔐 Secrets

| Secret File | Contains |
|-------------|----------|
| `telegram-bot-secret.yaml` | Telegram bot token + chat ID |
| `grafana-admin-secret.yaml` | Grafana admin username/password |

> ⚠️ These files should be encrypted (e.g., with **Sealed Secrets** or **SOPS**) before being committed to a real production GitOps repository.

---

## 🚀 Quick Start

### 1. Ensure the `monitoring` namespace exists

```bash
kubectl apply -f Kubernetes-Manifests-file/namespaces/monitoring.yaml
```

### 2. Let ArgoCD sync the monitoring Application

The monitoring stack is deployed automatically once `app-of-apps.yaml` is applied (see main [README.md](./README.md)). To sync manually:

```bash
argocd app sync monitoring
```

### 3. Verify the stack is running

```bash
kubectl -n monitoring get pods
```

Expected pods:
- `prometheus-kube-prometheus-stack-prometheus-*`
- `alertmanager-kube-prometheus-stack-alertmanager-*`
- `kube-prometheus-stack-grafana-*`

### 4. Access Grafana

```bash
kubectl -n monitoring port-forward svc/kube-prometheus-stack-grafana 3000:80
```

### 5. Test Telegram Alerting

Trigger a test alert (e.g., scale a deployment to 0 replicas temporarily) and confirm the notification arrives in the configured Telegram chat.

---

<div align="center">

**Monitoring Stack — MERN Stack DevOps End-to-End Project**

<img src="https://img.shields.io/badge/Metrics-Prometheus-E6522C?style=flat-square"/>
<img src="https://img.shields.io/badge/Dashboards-Grafana-F46800?style=flat-square"/>
<img src="https://img.shields.io/badge/Alerts-Telegram-26A5E4?style=flat-square"/>
<img src="https://img.shields.io/badge/GitOps-ArgoCD-orange?style=flat-square"/>

</div>
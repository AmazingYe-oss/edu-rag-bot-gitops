# edu-rag-bot-gitops (ArgoCD GitOps Configuration Repository)

![ArgoCD](https://img.shields.io/badge/GitOps-ArgoCD-orange?logo=argo)
![Kubernetes](https://img.shields.io/badge/Kubernetes-Ready-blue?logo=kubernetes)
![Kustomize](https://img.shields.io/badge/Kustomize-Config-blue)
![DevSecOps](https://img.shields.io/badge/Security-Trivy_Scanned-success)

## Repository Overview

This repository serves as the **GitOps State Repository** for the Cloud-Native RAG (Retrieval-Augmented Generation) Engineering Practice project.

Adhering strictly to GitOps best practices, this project implements a complete separation of concerns between application source code and infrastructure configuration. This repository contains zero business logic code; it exclusively hosts the declarative Kubernetes resource manifests (YAMLs) and ArgoCD application definitions.

Main Application Source Code Repository: [https://github.com/AmazingYe-oss/edu-rag-bot.git]

---

## Delivery Workflow

Acting as the single source of truth for the "Desired State", this repository sits downstream in the automated delivery pipeline, enabling a pure pull-based deployment model:

1. **Upstream Trigger**: GitHub Actions (CI) in the application repository builds the Docker image and pushes it to Aliyun ACR.
2. **Automated Write-back**: The final CI step utilizes a bot account to automatically commit the new Image Tag to the `kustomization.yaml` in this repository.
3. **ArgoCD Watch**: ArgoCD, deployed inside the Kubernetes cluster, continuously monitors this repository for changes.
4. **State Reconciliation**: Upon detecting configuration drift, ArgoCD automatically initiates a Sync operation, pulling the latest images and performing zero-downtime rolling updates.

---

## Directory Structure

Evolving alongside the microservices architecture, the manifests have been upgraded to support frontend-backend decoupling, Layer-7 traffic routing, and state persistence:

```text
├── apps/
│   └── edu-rag-bot/                   # Kubernetes manifests for the RAG bot
│       ├── kustomization.yaml         # Kustomize engine config (manages resources and image tags)
│       ├── deployment.yaml            # Stateless microservices (Frontend UI & Backend API)
│       ├── service.yaml               # Internal service exposure & Prometheus ServiceMonitor
│       ├── ingress.yaml               # Nginx Ingress Layer-7 routing rules
│       └── chromadb.yaml              # Stateful services (StatefulSet) & PVC mapping
├── edu-rag-bot-application.yaml       # ArgoCD Application root definition
└── README.md
```

---

## Engineering Highlights

1. **Microservices & Observability**: Manages Frontend and Backend independently via decoupled Deployments. Integrates `ServiceMonitor` to enable Prometheus auto-discovery and metrics scraping.
2. **Compute & Storage Separation**: Utilizes `StatefulSet` to deploy the ChromaDB vector database, dynamically provisioning Persistent Volume Claims (PVC) to guarantee zero data loss and sub-second knowledge base recovery upon Pod restarts.
3. **Declarative API Gateway**: Centralizes domain exposure and routing strategies through Nginx Ingress Controller.
4. **Native Kustomize Integration**: Abandons cumbersome global string replacements in favor of Kustomize's native capabilities for namespace injection and precise image tag overriding.

---

## Security & DevSecOps Notice

1. **Credential Isolation**: To ensure cloud-native security, this repository contains **NO plaintext sensitive credentials** (e.g., API Keys, Passwords). Production credentials are dynamically injected into container environments via Kubernetes Secrets.
2. **Shift-Left Security**: Both the infrastructure-as-code and container images are integrated with Trivy vulnerability scanners, establishing a robust DevSecOps loop to block critical vulnerabilities from entering the production cluster.

---
*Generated and maintained by automated GitOps workflows.*

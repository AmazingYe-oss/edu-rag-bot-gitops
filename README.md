# edu-rag-bot-gitops (ArgoCD GitOps Configuration Repository)

![ArgoCD](https://img.shields.io/badge/GitOps-ArgoCD-orange?logo=argo)
![Kubernetes](https://img.shields.io/badge/Kubernetes-Ready-blue?logo=kubernetes)
![Kustomize](https://img.shields.io/badge/Kustomize-Config-blue)
![DevSecOps](https://img.shields.io/badge/Security-Trivy_Scanned-success)

## Repository Overview

本仓库是 RAG 云原生工程化实践项目的 GitOps 状态配置仓库 (State Repository)。

根据云原生 GitOps 最佳实践，本项目严格实施了“应用源码与基础设施配置分离”的架构。本仓库中没有任何业务逻辑代码，仅包含用于 Kubernetes 部署的声明式资源清单 (YAML) 以及 ArgoCD 应用定义。

主应用源码仓库请访问: [https://github.com/AmazingYe-oss/edu-rag-bot.git]

---

## Delivery Workflow

本仓库处于整个自动化交付链路的下游，扮演着“期望状态声明 (Desired State)”的角色，并实现了真正的拉模式 (Pull-based) 交付：

1. 上游触发：主源码仓库中，GitHub Actions (CI) 完成镜像构建并推送至阿里云 ACR。
2. 自动回写：CI 流水线最后一步使用 bot 账号自动修改本仓库 kustomization.yaml 中的 Image Tag。
3. ArgoCD 纳管：Kubernetes 集群内部部署的 ArgoCD 实时监听本仓库的变更。
4. 状态调谐 (Reconciliation)：一旦发现本仓库的 YAML 发生漂移，ArgoCD 会自动向集群发起 Sync 操作，拉取最新镜像完成滚动更新。

---

## Directory Structure

随着微服务架构的演进，本仓库的资源清单已全面升级，支持前后端分离、七层流量路由与状态持久化：

```text
├── apps/
│   └── edu-rag-bot/                   # RAG 问答机器人 K8s 配置资源目录
│       ├── kustomization.yaml         # Kustomize 引擎配置 (统一管理资源与镜像版本控制)
│       ├── deployment.yaml            # 无状态微服务部署声明 (包含 Frontend UI 与 Backend API)
│       ├── service.yaml               # 内部服务暴露与 Prometheus ServiceMonitor 监控探针配置
│       ├── ingress.yaml               # Nginx Ingress 七层网关路由规则
│       └── chromadb.yaml              # 有状态服务声明 (StatefulSet) 及底层存储卷挂载 (PVC)
├── edu-rag-bot-application.yaml       # ArgoCD Application 根级资源定义
└── README.md
```

---

## Engineering Highlights

1. 微服务与可观测性：通过分离的 Deployment 独立管理前端与后端，并配置 ServiceMonitor 实现 Prometheus 自动发现与指标抓取。
2. 存储与计算分离：引入 StatefulSet 管理 ChromaDB 向量数据库，并动态申请 PVC 持久化数据，保障大模型知识库索引零丢失。
3. 声明式网关：通过 Ingress 资源统管微服务对外暴露域名与路由策略。
4. Kustomize 原生集成：摒弃繁杂的全局替换，利用 Kustomize 原生能力实现统一命名空间注入与精准的 Image Tag 覆盖。

---

## Security & DevSecOps Notice

1. 敏感凭证隔离：为保障云原生安全，本仓库不包含任何明文敏感凭证 (API Keys, Passwords 等)。生产环境中的实际凭证通过集群内部的 Kubernetes Secret 动态注入。
2. 安全左移 (Shift-Left)：基础架构代码与容器镜像均已集成 Trivy 安全扫描工具，构建完整的 DevSecOps 闭环，阻断高危漏洞流入生产集群。

---
*Generated and maintained by automated GitOps workflows.*

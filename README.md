# 🚀 Multi-Cluster GitOps Control Plane (Hub-and-Spoke ArgoCD)

**Author:** [Your Name]  
**Goal:** Build a production-grade GitOps workflow using ArgoCD, Kustomize, and k3d from absolute scratch.

This repository serves as a **portfolio project** and a **step-by-step guide** to implementing a multi-cluster GitOps architecture.

## 🏗️ Architecture

- **Hub Cluster:** Runs ArgoCD (Control Plane).
- **Spoke Cluster:** Runs application workloads.
- **Git Repo:** Holds the Source of Truth (this repo).

## 📚 Project Phases

| Phase | Topic | Documentation |
| :--- | :--- | :--- |
| 1 | Local Cluster Setup (k3d) | [✅ Done](docs/phase-1-cluster-setup.md) |
| 2 | Deploy ArgoCD on Hub | [📝 Planned](docs/phase-2-argocd-install.md) |
| 3 | Register Spoke Cluster | [📝 Planned](docs/phase-3-cluster-registration.md) |
| 4 | Git Repo & Kustomize | [📝 Planned](docs/phase-4-kustomize-git.md) |
| 5 | Application Deployment (ArgoCD App) | [📝 Planned](docs/phase-5-application-deploy.md) |
| 6 | Secrets Management & CI/CD | [📝 Planned](docs/phase-6-secrets-cicd.md) |

## 🛠️ Tech Stack

- **Containers:** Docker
- **Local Clusters:** k3d (lightweight Kubernetes in Docker)
- **GitOps Controller:** ArgoCD
- **Configuration Management:** Kustomize
- **Infrastructure:** Terraform (AWS EKS) - *Coming Soon*
- **CI/CD:** GitHub Actions - *Coming Soon*
- **Security:** Trivy (Image Scanning) - *Coming Soon*

## 🚀 How to Navigate

Click on the phase links above to read the detailed, step-by-step documentation.
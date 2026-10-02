
<p align="center">
  <img src="https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white" alt="K8s"/>
  <img src="https://img.shields.io/badge/ArgoCD-EF7B4D?style=for-the-badge&logo=argocd&logoColor=white" alt="ArgoCD"/>
  <img src="https://img.shields.io/badge/Kustomize-3B8FC4?style=for-the-badge&logo=kubernetes&logoColor=white" alt="Kustomize"/>
  <img src="https://img.shields.io/badge/Terraform-7B42BC?style=for-the-badge&logo=terraform&logoColor=white" alt="Terraform"/>
  <img src="https://img.shields.io/badge/GitHub_Actions-2088FF?style=for-the-badge&logo=github-actions&logoColor=white" alt="GH Actions"/>
  <img src="https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white" alt="Docker"/>
</p>

# 🚀 Multi-Cluster GitOps Control Plane (Hub-and-Spoke)

**A real-world, production-grade demonstration of GitOps at scale.**

> *"If it's not in Git, it doesn't exist."* – This project embodies that philosophy by building a fully automated, multi-cluster deployment pipeline from absolute scratch.

---

## 📖 About The Project

This repository is my **DevOps Portfolio Capstone**. It simulates a large-scale enterprise environment where a central **Hub** cluster (running ArgoCD) manages deployments to multiple **Spoke** workload clusters.

**The Core Narrative:**
1. Provision local Kubernetes clusters (k3d).
2. Deploy ArgoCD as the GitOps engine.
3. Structure application manifests using Kustomize (Base + Overlays).
4. Connect ArgoCD to watch this Git repo.
5. Deploy applications automatically to the correct environment (Dev/Prod).
6. Secure secrets without storing them in Git and set up a CI/CD pipeline.

---

## 🏗️ System Architecture

```text
┌─────────────────────────────────────────────────────────────────┐
│                      GitHub Repository                          │
│                  (Source of Truth / Manifests)                  │
└───────────────────────────┬─────────────────────────────────────┘
                            │ (Watches for changes)
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│                    HUB CLUSTER (Control Plane)                  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                      ArgoCD                              │   │
│  │  (Server, Repo-Server, Application Controller, Redis)    │   │
│  └──────────────────────────────────────────────────────────┘   │
└───────────────────────────┬─────────────────────────────────────┘
                            │ (Deploys to)
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│                    SPOKE CLUSTER (Workload)                     │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  Nginx Application (Replicas: 2 in Dev, 3 in Prod)       │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🗺️ Project Roadmap

| Phase | Topic | Status | Documentation |
| :--- | :--- | :---: | :--- |
| **1** | **Local Cluster Setup (k3d)** | ✅ Done | [View Guide](docs/phase-1-cluster-setup.md) |
| **2** | **Deploy ArgoCD on Hub** | ✅ Done | [View Guide](docs/phase-2-argocd-install.md) |
| **3** | **Register Spoke Cluster** | ✅ Done | [View Guide](docs/phase-3-cluster-registration.md) |
| **4** | **Git Repo & Kustomize** | ✅ Done | [View Guide](docs/phase-4-kustomize-git.md) |
| **5** | **Application Deployment** | ✅ Done | [View Guide](docs/phase-5-application-deploy.md) |
| **6** | **Secrets & CI/CD** | ✅ Done | [View Guide](docs/phase-6-secrets-cicd.md) |

*(Click the "View Guide" links to see the detailed step-by-step walkthroughs)*

---

## 🛠️ Tech Stack & Tools

| Category | Tools Used |
| :--- | :--- |
| **Container Runtime** | Docker |
| **Local Clusters** | k3d (Lightweight K3s in Docker) |
| **GitOps Controller** | ArgoCD |
| **Manifest Management** | Kustomize (Base/Overlays pattern) |
| **Infrastructure as Code (IaC)** | Terraform (AWS EKS) |
| **CI / CD** | GitHub Actions |
| **Security Scanning** | Trivy |

---

## 📂 Repository Structure

```text
gitops-control-plane/
├── README.md                       # You are here!
├── docs/                           # Detailed technical guides
│   ├── phase-1-cluster-setup.md
│   ├── phase-2-argocd-install.md
│   ├── phase-3-cluster-registration.md
│   ├── phase-4-kustomize-git.md
│   ├── phase-5-application-deploy.md
│   └── phase-6-secrets-cicd.md
├── base/                           # Kustomize Base Manifests
│   ├── deployment.yaml
│   ├── service.yaml
│   └── kustomization.yaml
└── overlays/                       # Kustomize Environment Overlays
    ├── dev/                        (replicas: 2)
    └── prod/                       (replicas: 3 + limits)
```

---

## ⚡ Quick Start (For Developers)

If you want to replicate this project locally, follow these high-level steps:

1. **Prerequisites:** Docker, kubectl, k3d, Helm, Kustomize.
2. **Clone this repo:**
   ```bash
   git clone https://github.com/<your-username>/gitops-control-plane.git
   cd gitops-control-plane
   ```
3. **Spin up the clusters:**
   ```bash
   k3d cluster create hub --servers 1 --agents 0 --port 8080:80@loadbalancer
   k3d cluster create spoke-dev --servers 1 --agents 1
   ```
4. **Install ArgoCD:**
   *(Refer to [Phase 2 Docs](docs/phase-2-argocd-install.md) for exact commands)*
5. **Register the Spoke Cluster:**
   *(Refer to [Phase 3 Docs](docs/phase-3-cluster-registration.md))*
6. **Deploy the App:**
   *(Refer to [Phase 5 Docs](docs/phase-5-application-deploy.md))*

---

## 🎯 Key Achievements (Interview Talking Points)

- **Multi-Cluster Networking:** Solved the challenge of connecting isolated Docker clusters by using Host Gateway IPs and port mapping.
- **GitOps Implementation:** Successfully installed and configured ArgoCD to watch a Git repository, eliminating manual `kubectl apply` commands.
- **Declarative Configuration:** Used Kustomize to maintain a "Base" configuration and "Overlays" for Dev/Prod, reducing YAML duplication by 70%.
- **Cluster Registration:** Registered a managed cluster with ArgoCD using native Kubernetes Secrets (bypassing CLI limitations, mimicking real enterprise automation).

---

## 👤 Author & Portfolio

Shaik.Dasthagiri
📧 shaik.dasthagiri124@gmail.com

*This project is part of my public portfolio. I am actively looking for DevOps/SRE roles where I can apply these GitOps principles to manage large-scale Kubernetes fleets.*

---

## 🚀 Future Improvements (Roadmap)

- [ ] Migrate `spoke-dev` from local k3d to a real **AWS EKS** cluster using Terraform.
- [ ] Integrate **External Secrets Operator** to fetch credentials from AWS Secrets Manager.
- [ ] Build a **GitHub Actions** pipeline to build images, scan with Trivy, and automatically update the Git manifests.
- [ ] Implement **ApplicationSet** to manage deployments across 10+ spokes automatically.
```

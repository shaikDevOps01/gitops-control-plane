## ✅ Phase 2 – Documentation Content

Great! Now we add the documentation for installing ArgoCD on the Hub cluster.

---

### 📝 What to do now

You will:
1. **Update** the root `README.md` (mark Phase 2 as "Done").
2. **Create** the new file `docs/phase-2-argocd-install.md` with the detailed content.
3. **Commit and Push** to GitHub.

---

### File 1: Update `README.md` (Root)

**Replace** the entire content of `README.md` with this updated version (Phase 2 is now marked as Done):

```markdown
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
| 2 | Deploy ArgoCD on Hub | [✅ Done](docs/phase-2-argocd-install.md) |
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
```

---

### File 2: Create `docs/phase-2-argocd-install.md`

Create this new file and paste the content below:

```markdown
# Phase 2: Deploy ArgoCD on Hub Cluster

**Date:** June 2026  
**Objective:** Install ArgoCD on the `hub` cluster using Helm, access the UI, and log in as admin.

## 🎯 Why we did this

ArgoCD is the heart of GitOps. It watches your Git repository and automatically syncs your applications to the target clusters. We deploy it on the `hub` cluster because:

- It provides a central control plane.
- It keeps the workload clusters (spokes) clean.
- It follows the "separation of concerns" principle.

## 🖥️ Commands Executed

### 1. Add the ArgoCD Helm repository
```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
```

### 2. Create a namespace and install ArgoCD
```bash
kubectl create namespace argocd

helm install argocd argo/argo-cd \
  --namespace argocd \
  --set server.service.type=ClusterIP \
  --set configs.params."server\.insecure"=true
```

> 💡 **Why `server.insecure=true`?** This allows HTTP access (no TLS). It is only used for local development. In production, you would configure HTTPS with proper certificates.

### 3. Wait for all pods to become `Running`
```bash
kubectl get pods -n argocd -w
```
Press `Ctrl+C` once all pods show `1/1 Running`. You should see:
- `argocd-server`
- `argocd-repo-server`
- `argocd-application-controller`
- `argocd-redis`
- `argocd-dex-server`
- `argocd-notifications-controller`
- `argocd-applicationset-controller`

### 4. Retrieve the initial admin password
```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d && echo
```
Copy the password output. You will need it to log in.

### 5. Port-forward to access the UI
```bash
kubectl port-forward svc/argocd-server -n argocd 9090:80
```
Now open your browser and go to: `http://localhost:9090`

Login with:
- **Username:** `admin`
- **Password:** (the password you copied)

## 🔥 Troubleshooting

### Issue: "connection reset by peer" or "connection refused"
**Root cause:** Forwarding to the pod (`pod/argocd-server-xxx`) on port 443 (HTTPS) caused TLS handshake issues.

**Fix:** We switched to forwarding the **Service** on port **80** (HTTP):
```bash
kubectl port-forward svc/argocd-server -n argocd 9090:80
```

### Issue: Pods stuck in `Pending` or `CrashLoopBackOff`
**Fix:** Restart all ArgoCD pods to clear the state:
```bash
kubectl delete pods --all -n argocd
```
Wait a minute and re-check `kubectl get pods -n argocd`.

## ✅ Phase 2 Checklist

- [x] ArgoCD installed on hub cluster using Helm.
- [x] All ArgoCD pods running.
- [x] Admin password retrieved from secret.
- [x] UI accessible via port-forward on `http://localhost:9090`.
- [x] Admin login successful (dashboard showing "No applications").

## 📝 Lessons Learned

- **Helm simplifies complex installations:** ArgoCD has 7+ components. Helm manages them as a single unit.
- **TLS can complicate local testing:** Using `server.insecure=true` and forwarding to HTTP (port 80) avoids certificate errors.
- **The admin password is stored in a Kubernetes secret:** It must be decoded with `base64 -d`.
```

---

`docs/phase-1-cluster-setup.md`:

```markdown
# Phase 1: Local Multi-Cluster Environment (k3d)

**Date:** June 2026  
**Objective:** Create two isolated Kubernetes clusters locally to simulate a Hub-and-Spoke architecture.

## 🎯 Why we did this

In production, ArgoCD (the controller) runs on a dedicated cluster, and it manages many "workload" clusters. We are simulating this locally:

- **`hub`:** Will host ArgoCD (Control Plane).
- **`spoke-dev`:** Will run our applications (Managed Cluster).

## 🖥️ Commands Executed

### 1. Create the clusters
```bash
# Hub cluster (1 control-plane node, 0 workers, map port 8080 for UI)
k3d cluster create hub --servers 1 --agents 0 --port 8080:80@loadbalancer

# Spoke cluster (1 control-plane, 1 worker)
k3d cluster create spoke-dev --servers 1 --agents 1
```

### 2. Verify clusters are running
```bash
k3d cluster list
```
**Expected Output:**
```
NAME        SERVERS   AGENTS   LOADBALANCER
hub         1/1       0/0      true
spoke-dev   1/1       1/1      true
```

### 3. Test context switching
```bash
# Switch to hub
kubectl config use-context k3d-hub
kubectl get nodes

# Switch to spoke
kubectl config use-context k3d-spoke-dev
kubectl get nodes
```

## 🔥 Critical Networking Discovery

k3d creates **separate Docker networks** for each cluster. To allow the Hub to talk to the Spoke, we had to find the Hub's gateway IP and the Spoke's host port.

### Getting the Endpoint
```bash
# Get the gateway IP of the hub network
docker network inspect k3d-hub | grep Gateway
# Output: "Gateway": "172.18.0.1"

# Get the host port mapped to the spoke's API (6443)
docker ps | grep spoke-dev-serverlb | grep -oP '0.0.0.0:\d+->6443'
# Output: 0.0.0.0:35327->6443
```

### Testing Connectivity
```bash
# From the Ubuntu host, test the spoke API
curl -k https://172.18.0.1:35327/version
```
**Result:** Received a `401 Unauthorized` response.

> 💡 **Why 401 is a success:** It proves the network route works! The Spoke's Kubernetes API is rejecting the request because we haven't provided authentication yet (which ArgoCD will do later).

## ✅ Phase 1 Checklist

- [x] Hub cluster created and running.
- [x] Spoke cluster created and running.
- [x] Network connectivity verified (401 response received).
- [x] Context switching (`kubectl config use-context`) working.

## 📝 Lessons Learned

- Kubernetes clusters are **isolated by default**, even in Docker.
- To connect them locally, you must use the **Docker gateway IP** and the **host-mapped ports**.
- A `401 Unauthorized` error is a positive sign during network debugging—it means the API is reachable!
```

---

### ⚙️ Step 5: Push Phase 1 to GitHub

Now run these commands to commit and push your first changes:

```bash
cd ~/gitops-control-plane

# Add files to Git
git add README.md docs/phase-1-cluster-setup.md

# Commit with a descriptive message
git commit -m "docs: Add Phase 1 documentation (k3d cluster setup)"

# Set the remote origin (replace with your URL)
git remote add origin https://github.com/<your-username>/gitops-control-plane.git

# Push to GitHub
git branch -M main
git push -u origin main
```

---

### 🔍 Check Your Work

1. Open your browser and go to `https://github.com/<your-username>/gitops-control-plane`.
2. You should see the `README.md` rendered beautifully.
3. Click into the `docs/` folder. You should see `phase-1-cluster-setup.md`.

---

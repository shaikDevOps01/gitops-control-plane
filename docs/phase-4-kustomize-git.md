# Phase 4: Git Repo Structure & Kustomize

**Date:** June 2026  
**Objective:** Create the "Source of Truth" Git repository and use Kustomize to manage environment configurations (Dev vs Prod) without duplicating YAML files.

## 🎯 Why we did this

In GitOps, the Git repository is the single source of truth. We need to structure our manifests so that:
- We don't repeat ourselves (DRY principle).
- We can easily promote changes from Dev to Prod.
- ArgoCD can watch specific folders for changes.

## 🧠 The Kustomize Concept

- **Base:** The common configuration (e.g., Nginx deployment with 1 replica).
- **Overlays:** Environment-specific patches (e.g., Dev gets 2 replicas, Prod gets 3 replicas + resource limits).

## 📂 Repository Structure Created

```text
gitops-control-plane/
├── base/
│   ├── deployment.yaml      (nginx:1.25, replicas: 1)
│   ├── service.yaml
│   └── kustomization.yaml
└── overlays/
    ├── dev/
    │   ├── deployment-patch.yaml (replicas: 2)
    │   └── kustomization.yaml
    └── prod/
        ├── deployment-patch.yaml (replicas: 3, limits)
        └── kustomization.yaml
```

## 🖥️ Commands Executed

### 1. Create the base manifests
```bash
mkdir -p base overlays/dev overlays/prod

cat > base/deployment.yaml <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
EOF

cat > base/service.yaml <<EOF
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP
EOF

cat > base/kustomization.yaml <<EOF
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- deployment.yaml
- service.yaml
EOF
```

### 2. Create the Dev overlay
```bash
cat > overlays/dev/kustomization.yaml <<EOF
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- ../../base
patches:
- path: deployment-patch.yaml
EOF

cat > overlays/dev/deployment-patch.yaml <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 2
EOF
```

### 3. Create the Prod overlay
```bash
cat > overlays/prod/kustomization.yaml <<EOF
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- ../../base
patches:
- path: deployment-patch.yaml
EOF

cat > overlays/prod/deployment-patch.yaml <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: nginx
        image: nginx:1.26
        resources:
          requests:
            memory: "128Mi"
            cpu: "250m"
          limits:
            memory: "256Mi"
            cpu: "500m"
EOF
```

### 4. Test Kustomize builds
```bash
kustomize build overlays/dev    # Should show 2 replicas
kustomize build overlays/prod   # Should show 3 replicas, limits, and 1.26 image
```

### 5. Push to GitHub
```bash
git init
git add .
git commit -m "Add base and overlays"
git remote add origin https://github.com/<your-username>/gitops-control-plane.git
git push -u origin main
```

## 🔥 Troubleshooting

| Issue | Fix |
| :--- | :--- |
| `kustomize: command not found` | Install it: `curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh" | bash` and move to `/usr/local/bin`. |
| YAML indentation errors | Use 2 spaces for indentation. Check your patch files for missing `spec:` fields. |

## 📝 Lessons Learned

- **Kustomize avoids "YAML sprawl":** Instead of copying 50 lines of YAML for Dev and Prod, we only write the 2 lines that change.
- **Base + Overlays is a clean pattern:** It allows you to test changes in Dev before promoting to Prod by simply changing the overlay.

## ✅ Phase 4 Checklist

- [x] Base manifests created (Deployment, Service).
- [x] Dev overlay created (replicas: 2).
- [x] Prod overlay created (replicas: 3, limits, 1.26 image).
- [x] `kustomize build` tested successfully.
- [x] Code committed and pushed to GitHub.
```

---

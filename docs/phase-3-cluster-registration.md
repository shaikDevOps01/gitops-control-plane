## ✅ Yes, your understanding is 100% correct

The **root `README.md`** is the **only** main landing page for your repository. 
You do NOT need to create separate `README.md` files inside the `docs/` folder. 
Instead, inside `docs/`, you just create the specific **detailed guide files** (like `phase-3-cluster-registration.md`). The root `README.md` links to these files perfectly. 

**You are done editing the root README forever.** Now we just build out the detailed guides in the `docs/` folder.

---

## 🚀 Phase 3 – Documentation Content

Here is the content for `docs/phase-3-cluster-registration.md`. 
Copy this and paste it into a new file in your `docs/` folder.

### File: `docs/phase-3-cluster-registration.md`

```markdown
# Phase 3: Register Spoke Cluster with ArgoCD

**Date:** June 2026  
**Objective:** Connect the `spoke-dev` cluster to ArgoCD (running on the `hub`) so that ArgoCD can deploy applications to it.

## 🎯 Why we did this

ArgoCD runs on the Hub, but it is useless unless it knows where to deploy the applications. This phase establishes the **trust relationship** between the Hub and the Spoke. We generate a service account token on the Spoke and store it as a secret on the Hub.

## 🖥️ Commands Executed

### 1. Switch to the Spoke cluster context
```bash
kubectl config use-context k3d-spoke-dev
```

### 2. Create a Service Account and bind it to `cluster-admin`
```bash
# Create namespace for ArgoCD's cross-cluster access
kubectl create namespace argocd

# Create the service account
kubectl create serviceaccount argocd-manager -n argocd

# Give it admin permissions (for demo purposes)
kubectl create clusterrolebinding argocd-manager-binding \
  --clusterrole=cluster-admin \
  --serviceaccount=argocd:argocd-manager
```

### 3. Generate a long-lived token
Since Kubernetes 1.24+, service accounts do not automatically have token secrets. We create one manually.

```bash
kubectl apply -f - <<YAML
apiVersion: v1
kind: Secret
metadata:
  name: argocd-manager-token
  namespace: argocd
  annotations:
    kubernetes.io/service-account.name: argocd-manager
type: kubernetes.io/service-account-token
YAML

# Wait a moment, then extract the token
kubectl get secret argocd-manager-token -n argocd -o jsonpath="{.data.token}" | base64 -d && echo
```

> 💡 **Important:** Copy the long token string that is printed (starts with `eyJ...`). You will need it in the next step.

---

### 4. Switch back to the Hub cluster
```bash
kubectl config use-context k3d-hub
```

### 5. Create the Cluster Secret on the Hub
Instead of using the `argocd cluster add` CLI (which can be flaky), we create the Kubernetes secret directly. ArgoCD watches for secrets with the label `argocd.argoproj.io/secret-type: cluster`.

*Replace the placeholders:*
- `GATEWAY_IP`: Get it via `docker network inspect k3d-hub | grep Gateway`.
- `SPOKE_PORT`: Get it via `docker ps | grep spoke-dev-serverlb`.
- `TOKEN`: Paste the token from Step 3.

```bash
kubectl apply -f - <<YAML
apiVersion: v1
kind: Secret
metadata:
  name: spoke-dev-cluster
  namespace: argocd
  labels:
    argocd.argoproj.io/secret-type: cluster
type: Opaque
stringData:
  name: spoke-dev
  server: https://172.18.0.1:35327
  config: |
    {
      "bearerToken": "PASTE_YOUR_TOKEN_HERE",
      "tlsClientConfig": {
        "insecure": true
      }
    }
YAML
```

### 6. Verification
- **Via UI:** Open ArgoCD UI (`http://localhost:9090`) → Settings → Clusters. You should see `spoke-dev` with a **"Successful"** connection status.
- **Via CLI:** 
  ```bash
  kubectl get secret -n argocd -l argocd.argoproj.io/secret-type=cluster
  ```

## 🔥 Troubleshooting

| Issue | Likely Cause | Fix |
| :--- | :--- | :--- |
| **Connection Refused** | Wrong IP or Port. | Re-run `docker network inspect` and `docker ps` to get the fresh Gateway IP and port. |
| **Unauthorized / 403** | Token expired or incorrect. | Re-generate the token on the spoke cluster and update the secret on the hub. |
| **Cluster not appearing in UI** | ArgoCD controller needs a restart. | Run `kubectl delete pods -n argocd -l app.kubernetes.io/name=argocd-application-controller` to force a refresh. |

## 📝 Lessons Learned

- **Skip the CLI, use Secrets:** Creating the `Secret` manually is more reliable than `argocd cluster add`. It is also **the preferred method in Infrastructure-as-Code** (Terraform) because you can just `kubectl apply -f` the YAML.
- **Service Account Tokens are powerful:** The token we created has `cluster-admin` rights. In production, you would scope this down to specific namespaces.

## ✅ Phase 3 Checklist

- [x] Service account `argocd-manager` created on spoke.
- [x] ClusterRoleBinding created for admin rights.
- [x] Long-lived token generated.
- [x] Cluster secret created on the hub with correct label.
- [x] Spoke cluster shows "Successful" in ArgoCD UI.
```

---

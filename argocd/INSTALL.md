# Installing Argo CD

This repo assumes a running Kubernetes cluster with Argo CD installed. Argo CD
itself is **not** vendored here — install it once, then let the app-of-apps
root take over.

## 1. Install Argo CD

```bash
kubectl create namespace argocd
kubectl apply -n argocd \
  -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Wait for the pods to become ready:

```bash
kubectl -n argocd wait --for=condition=available deploy --all --timeout=300s
```

## 2. Apply the AppProject

The project scopes which repo and namespaces the demo apps may use:

```bash
kubectl apply -f argocd/projects/appproject.yaml
```

## 3. Apply the root (app-of-apps) Application

```bash
kubectl apply -f apps/argocd/root-app.yaml
```

Argo CD then reconciles the root app, which discovers the child Applications
under `apps/applications/` and deploys the dev and prod overlays.

## 4. (Optional) Access the UI

```bash
kubectl -n argocd port-forward svc/argocd-server 8080:443
# initial admin password:
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath='{.data.password}' | base64 -d; echo
```

> Adjust `repoURL` in every Argo CD manifest to point at **your** fork before
> applying, otherwise Argo CD will try to sync from this repo's placeholder URL.

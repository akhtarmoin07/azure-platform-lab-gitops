# Bootstrap Argo CD

Argo CD is the only cluster component installed imperatively. After bootstrap,
Argo CD reconciles platform and application configuration from this repository.

## Prerequisites

- The current `kubectl` context is `aks-azplab-lab`.
- The operator is a member of the platform administrators Entra group.
- Helm and kubelogin are installed.

## Validate

```bash
helm dependency update bootstrap/argocd
helm lint bootstrap/argocd
helm template argocd bootstrap/argocd --namespace argocd > /tmp/argocd-rendered.yaml
```

Commit the generated `bootstrap/argocd/Chart.lock`. The downloaded `charts/`
directory is intentionally ignored.

## Install or upgrade

```bash
helm upgrade --install argocd bootstrap/argocd \
  --namespace argocd \
  --create-namespace \
  --wait \
  --timeout 10m
```

## Verify

```bash
helm status argocd --namespace argocd
kubectl get pods --namespace argocd
kubectl get services --namespace argocd
```

The Argo CD API/UI service remains private to the cluster through a `ClusterIP`
service. Use port forwarding for lab access instead of provisioning a public
load balancer.

## Hand control to GitOps

After Argo CD is healthy, apply the root application once:

```bash
kubectl apply -f bootstrap/root-application.yaml
kubectl get applications,appprojects --namespace argocd
```

From this point onward, change cluster configuration by pull request. The root
application reconciles the child applications declared under `clusters/lab`.

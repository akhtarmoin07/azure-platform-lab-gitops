# Azure Platform Lab GitOps

Declarative Kubernetes and Helm configuration reconciled by Argo CD across
development and production-pattern namespaces in the shared AKS lab cluster.

## Ownership

- The platform team owns Argo CD bootstrap, cluster guardrails and shared services.
- Application teams contribute environment configuration through pull requests.
- Production-pattern changes require review before they merge.

## Repository layout

```text
bootstrap/argocd/  Pinned Helm package used for the one-time Argo CD bootstrap
docs/              Bootstrap and operational documentation
```

Environment baselines and application definitions will be added after Argo CD is
installed and verified.

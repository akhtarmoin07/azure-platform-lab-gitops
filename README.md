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
clusters/lab/      Root Argo CD projects and child application declarations
platform/          Namespace guardrails owned by the platform team
docs/              Bootstrap and operational documentation
```

The lab currently runs `dev` and `prod` on one shared worker node. Namespace
RBAC, quotas, Pod Security Admission and default-deny network policies provide
logical isolation; they do not provide node-level high availability.

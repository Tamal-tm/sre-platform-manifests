# sre-platform-manifests

> Part of a 4-repo GitOps setup — see [sre-platform](https://github.com/Tamal-tm/sre-platform#repo-structure) for the full repo map and architecture.

Kubernetes manifests for the Self-Healing Observable Platform. This is the
**deployment source of truth** — ArgoCD continuously watches this repo and
syncs the live cluster to match whatever is committed here. Nothing gets
deployed by running `kubectl apply` by hand; it gets deployed by pushing
here and letting ArgoCD reconcile.

## Repo structure

\`\`\`
sre-platform-manifests/
├── argocd.yaml                  # ArgoCD Application definition — tells
│                                 # ArgoCD to watch this repo's base/ path
│                                 # and sync it into the cluster
└── base/
    ├── namespace.yaml            # sre-platform namespace
    ├── configmap.yaml            # app-level configuration
    ├── service-a-deployment.yaml
    ├── service-a-service.yaml
    ├── service-b-deployment.yaml
    ├── service-b-service.yaml
    ├── service-monitors.yaml     # ServiceMonitor CRDs — tells Prometheus
    │                             # how to discover and scrape service-a/b
    └── slo-alerts.yaml           # PrometheusRule — the HighErrorBudgetBurn
                                  # burn-rate alert definition
\`\`\`

## Why this repo exists separately

In a GitOps setup, **desired state** (what *should* be running) has to live
somewhere Git-tracked and independent of the application code itself. If
manifests lived inside `sre-platform-app` alongside the source code, a
routine app change and a deployment-config change would be indistinguishable
in git history, and ArgoCD would have no clean way to reconcile drift
against "what's actually meant to be live." Splitting it out means:

- ArgoCD's `Application` object watches **this repo only** — application
  code changes in `sre-platform-app` have zero effect on the cluster until
  someone (or a CI pipeline) updates the manifest here to reference the new
  image tag.
- Any manual `kubectl` change made directly against the cluster

# Manifestos handler

## Mongo

Optional: use this as a template to register the MongoDB app in Argo CD.
Apply with: kubectl apply -f apps/mongodb/ARGOCD_APPLICATION.yaml
Or create the Application via the Argo CD UI with the same spec. Argo CD will detect Chart.yaml in apps/mongodb and use Helm
runs:

```bash
"helm dependency build" + "helm template" with values.yaml.

```

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-mongodb
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/jotace-labs/k8s-argo-apps
    path: apps/mongodb
    targetRevision: HEAD
    helm:
      releaseName: my-mongo
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

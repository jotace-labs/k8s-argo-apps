# k8s-argo-apps

Kubernetes manifests and Helm charts for apps managed by Argo CD.

## Repo structure

- **`apps/`** — Apps deployed with **Helm**. Each subfolder is a chart (or wrapper chart) with its own `Chart.yaml` and `values.yaml`. Argo CD detects `Chart.yaml` and runs `helm dependency build` + `helm template`.
  - `apps/mongodb/` — MongoDB (Bitnami chart), values and NodePort in `values.yaml`.

- **`apps-kustomize/`** — Apps deployed with **Kustomize**. Each app has a `base/` (shared manifests) and `overlay/<env>/` (e.g. `dev`) that patches namespace, namePrefix, and env-specific settings. Useful for patching diffs (e.g. `apps-kustomize/my-producer/overlay/dev`).
  - `my-producer/` — Kafka producer.
  - `my-consumer/` — Kafka consumer.
  - `redpanda/` — Redpanda (Kafka API), with PVC, Deployment, Service, and NodePort.

---

## Mongo

Repos registered as source: `https://charts.bitnami.com/bitnami`

Optional: it possible to use this as a template to register the MongoDB app in Argo CD:

```bash
kubectl apply -f {ARGOCD_APPLICATION}.yaml
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

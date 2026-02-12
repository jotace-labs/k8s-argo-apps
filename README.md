# k8s-argo-apps

Kubernetes manifests and Helm charts for apps managed by Argo CD.

## App

Simple cluster that has:

- redpanda instance
- a mongodb instance (as helm chart)
- a producer that produces records every 30s
- a consumer that consumes all records and inserts it in mongo in order  

## Pipeline

### Buildign apps

Using as example: [Producer App](https://github.com/jotace-labs/my-producer) -> service that produces a kafka record every 30s.

Every release on the apps triggers this workflow:

```bash
name: Build Docker image
on:
  push:
    tags: ["v*.*.*"]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Log in to ghcr.io
        run: |
          echo "${{ secrets.GHCR_TOKEN }}" | docker login ghcr.io -u ${{ github.actor }} --password-stdin

      - name: Build and tag image    
        run: |
          TAG=$(echo $GITHUB_SHA | cut -c1-7)
          docker build \
             --build-arg GITHUB_TOKEN=${{ secrets.PAT }} \
             --build-arg GITHUB_TOKEN_OWNER=${{ secrets.PAT_OWNER }} \
             -t ghcr.io/${{ github.repository }}:$TAG .

          echo "IMAGE_TAG=$TAG" >> $GITHUB_ENV
  
      - name: Push image to GHCR
        run: docker push ghcr.io/${{ github.repository }}:${{ env.IMAGE_TAG }}
```

It publishes the docker images as packages stored here:

[Jotace Labs Packages](https://github.com/orgs/jotace-labs/packages)

### Updating manifesto

Since im not hosting my cluster online (cloud), I didnt find a solution to automatically apply the manifestos changes in this repo based on actions. So far, we manually apply the image version in `apps/kustomize` and, if it's a new application at all, we have to manually create it the first time in ArgoCD. The rest, it automatically detecs commits to this `k8s-argo-apps` and apply changes. 

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

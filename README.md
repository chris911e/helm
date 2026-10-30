# helm

Helm charts for my home Kubernetes cluster, deployed with Argo CD (GitOps).
Each top-level directory is a standalone chart; `argocd/` holds the Argo CD
`Application` manifests that point at them.

## Charts

| Chart | What it runs | Namespace (dev) |
| --- | --- | --- |
| [`portfolio-frontend`](portfolio-frontend/) | The portfolio site image from `ghcr.io/chris911e/portfolio-frontend` | `dev` |
| [`cloudflared`](cloudflared/) | Cloudflare Tunnel connector (remotely managed tunnel) | `cloudflared` |

## Layout

```
argocd/
  dev/                 # Argo CD Applications for the dev environment
  prod/                # Argo CD Applications for prod (empty for now)
<chart>/
  Chart.yaml
  values.yaml          # base values
  values-dev.yaml      # dev overrides, merged on top of values.yaml
  values-prod.yaml     # prod overrides (portfolio-frontend only, for now)
  templates/
```

Each Argo CD `Application` renders its chart with `values.yaml` followed by the
environment file, syncs automatically (`prune` and `selfHeal`), and creates its
namespace if it is missing.

## Deploying

Register the Applications with Argo CD:

```sh
kubectl apply -f argocd/dev/
```

After that, Argo CD tracks `HEAD` of this repo, so changes are deployed by
pushing to `main`.

To release a new portfolio-frontend build, bump `image.tag` in
`portfolio-frontend/values-dev.yaml` and push.

## Required secrets

The charts read credentials from Secrets that are not stored in this repo.
Create them in each namespace before syncing:

| Namespace | Secret | Key | Used by |
| --- | --- | --- | --- |
| `dev` | `ghcr-pull` | (docker-registry) | image pulls |
| `cloudflared` | `cloudflared-token` | `token` | cloudflared tunnel token |

For example:

```sh
kubectl -n dev create secret docker-registry ghcr-pull \
  --docker-server=ghcr.io --docker-username=<user> --docker-password=<token>

kubectl -n cloudflared create secret generic cloudflared-token --from-literal=token=<token>
```

## Rendering locally

```sh
helm template portfolio-frontend ./portfolio-frontend \
  -f portfolio-frontend/values.yaml -f portfolio-frontend/values-dev.yaml
```

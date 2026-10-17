# Portfolio frontend Helm example

This chart deploys a small nginx web server by default. Set `image.repository`
and `image.tag` in `values.yaml` (or provide an Argo CD Helm values override)
to use your frontend container image.

The Argo CD `Application` is in `argocd/portfolio-frontend.yaml`. Apply it to
the namespace where Argo CD is installed:

```sh
kubectl apply -f argocd/portfolio-frontend.yaml
```

Argo CD will read this Git repository's `portfolio-frontend` directory, create
the `portfolio-frontend` destination namespace, and keep the release in sync.
The optional Ingress is disabled by default; set `ingress.enabled: true` and
configure its host for your cluster to expose the service externally.

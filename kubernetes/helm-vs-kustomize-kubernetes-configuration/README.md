# Helm vs Kustomize Kubernetes configuration

Companion example for [Helm vs Kustomize: Choose the Right Kubernetes Config](https://reclear.io/blog/helm-vs-kustomize-kubernetes-configuration).

This example renders the same small web application with Helm and Kustomize. Both production outputs use four replicas, the `production` namespace, and image `ghcr.io/example/web:1.4.3`.

## Tested versions

- Helm `3.17.1`
- Kubernetes CLI `1.32.2`
- Embedded Kustomize `5.5.0`
- Docker is not required because the example renders configuration only.
- A Kubernetes cluster is not required. The validation target uses local rendering and client-side dry runs.

## Prerequisites

Install Helm and kubectl, then verify the versions:

```bash
helm version --short
kubectl version --client
kubectl kustomize version
```

The commands should report versions compatible with the tested versions above. Minor version differences should not change the example output, but review rendered YAML after upgrades.

## Layout

```text
.
├── helm/web/
│   ├── Chart.yaml
│   ├── values.yaml
│   ├── values-production.yaml
│   └── templates/
└── kustomize/
    ├── base/
    └── overlays/production/
```

## Validate locally

Run the non-destructive validation target:

```bash
make validate
```

The target runs `helm lint`, renders both configurations, and checks the production values in the generated YAML. It does not contact a cluster or create resources.

To render each output manually:

```bash
kubectl kustomize kustomize/overlays/production > /tmp/web-production-kustomize.yaml
helm template prod helm/web \
  --namespace production \
  --values helm/web/values-production.yaml \
  > /tmp/web-production-helm.yaml
```

Inspect the rendered resources before using them in a deployment workflow. A real deployment should add policy checks, image provenance verification, and secret handling appropriate to the target cluster.

## Optional deployment

Only apply the rendered output after reviewing it and selecting a configured cluster:

```bash
kubectl apply -f /tmp/web-production-kustomize.yaml
kubectl rollout status deployment/prod-web --namespace production
```

The example references `ghcr.io/example/web:1.4.3`, which is a placeholder image. Replace it with an image that exists in your registry before deploying. The example does not create a namespace, registry credentials, ingress, or application Service health endpoint.

## Limitations and security notes

- The example demonstrates configuration rendering, not a production-ready application.
- The image reference is intentionally a placeholder and is not pulled during validation.
- Do not put production credentials in Helm values or Kustomize files. Use an approved secret-management workflow.
- Rendered YAML should be scanned with the policies used by your delivery platform before deployment.
- Pin chart dependencies, remote bases, images, and third-party actions in a production workflow.

## Cleanup

The validation target writes only temporary files under `.tmp/` and removes them when it completes. If you applied the optional manifest, remove the example resources with:

```bash
kubectl delete -f /tmp/web-production-kustomize.yaml --namespace production
```

Then remove the namespace only if it was created for this example and is not shared:

```bash
kubectl delete namespace production
```

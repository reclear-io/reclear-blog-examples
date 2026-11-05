.PHONY: validate clean

KUSTOMIZE_OUTPUT := .tmp/web-production-kustomize.yaml
HELM_OUTPUT := .tmp/web-production-helm.yaml

validate:
	@set -eu; \
	mkdir -p .tmp; \
	kubectl kustomize kustomize/overlays/production > $(KUSTOMIZE_OUTPUT); \
	helm lint helm/web; \
	helm template prod helm/web --namespace production --values helm/web/values-production.yaml > $(HELM_OUTPUT); \
	grep -q 'name: prod-web' $(KUSTOMIZE_OUTPUT); \
	grep -q 'namespace: production' $(KUSTOMIZE_OUTPUT); \
	grep -q 'replicas: 4' $(KUSTOMIZE_OUTPUT); \
	grep -q 'ghcr.io/example/web:1.4.3' $(KUSTOMIZE_OUTPUT); \
	grep -q 'name: prod-web' $(HELM_OUTPUT); \
	grep -q 'namespace: production' $(HELM_OUTPUT); \
	grep -q 'replicas: 4' $(HELM_OUTPUT); \
	grep -q 'ghcr.io/example/web:1.4.3' $(HELM_OUTPUT); \
	rm -rf .tmp; \
	echo "Helm and Kustomize validation passed."

clean:
	rm -rf .tmp

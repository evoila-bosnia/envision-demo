# OpenTelemetry Demo (reduced)

Deployed as an ArgoCD **Helm** Application — see `argocd/app-otel-demo.yaml`.
There is no kustomize base here on purpose: the demo is a large upstream Helm
chart; ArgoCD renders it in-cluster from the pinned chart version.

- Chart: `opentelemetry-demo` **0.42.0** (repo
  `https://open-telemetry.github.io/opentelemetry-helm-charts`), appVersion `3.1.0`.
- Reduced: bundled `grafana` / `prometheus` / `jaeger` / `opensearch` **disabled**
  (the LGTM all-in-one is the single observability backend); the demo's
  OpenTelemetry Collector exports OTLP to `otel-lgtm.envision-demo.svc:4317`.
- Still deployed: the ~15 microservices + `flagd` + `load-generator` (so real
  traces/metrics/logs flow into LGTM) + the demo `otel-collector` + `frontend-proxy`.

## If the two nested guest nodes are too small
Trim further in `app-otel-demo.yaml` helm values:
- `load-generator: { enabled: false }` (stops synthetic load; telemetry goes quiet)
- disable image-heavy optional services (e.g. `kafka`, `accounting`, `fraud-detection`,
  `image-provider`) via `components.<name>.enabled: false`.

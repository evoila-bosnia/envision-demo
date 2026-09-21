# envision-demo — Envision 2026 stand GitOps repo

The demo application + observability layer visitors see and drive at the Envision
2026 stand. Everything is deployed onto the **VKS guest cluster** in lab estate
**c3env1**, driven end-to-end through the **MEHO backplane** in the **envision
tenant**. ArgoCD (installed on the guest cluster) reconciles this repo.

## What's here
| Path | App | How ArgoCD deploys it |
|------|-----|-----------------------|
| `apps/podinfo/` | podinfo (stateless sample app) | kustomize |
| `apps/otel-lgtm/` | Grafana LGTM all-in-one (Grafana + Prometheus + Loki + Tempo) | kustomize |
| `apps/otel-demo/` | OpenTelemetry Demo (reduced) | Helm (chart `opentelemetry-demo` 0.42.0) |
| `overlays/envision/` | **The single place for the two Wave 2/3 unknowns** (StorageClass, LB) | kustomize Component |
| `argocd/` | ArgoCD `Application` / `AppProject` CRs | applied via `k8s.apply` |

## Pinned versions
- podinfo image `ghcr.io/stefanprodan/podinfo:6.15.0`
- otel-lgtm image `grafana/otel-lgtm:0.33.1`
- opentelemetry-demo Helm chart `0.42.0` (appVersion 3.1.0)
- ArgoCD install manifest `v3.3.9` (matches the `argocd-api-3.x` connector / `rdc-argocd`)

## The two unknowns (set ONCE, in `overlays/envision/`)
1. **StorageClass** — `overlays/envision/patch-storageclass.yaml`: replace
   `REPLACE_ME_STORAGE_CLASS` with the NFS-SPBM-policy-derived StorageClass the
   Supervisor propagates into the guest cluster (named explicitly; no default).
2. **LoadBalancer** — `overlays/envision/patch-loadbalancer.yaml`: Foundation LB,
   VIP auto-assigned by default; uncomment `loadBalancerIP` to pin one from the pool.

Both are consumed by every app overlay via the shared kustomize **Component**
(`apps/*/overlays/envision` → `components: [../../../../overlays/envision]`), so
there is exactly one edit site for each. The OpenTelemetry Demo (Helm) carries its
own LB knob (`components.frontend-proxy.service.type`) in `argocd/app-otel-demo.yaml`
and is stateless (no StorageClass needed).

## Namespaces
- `argocd` — ArgoCD control plane (installed via `k8s.apply`).
- `envision-demo` — all demo workloads (set by the overlay + `CreateNamespace=true`).

## meho-first
Nothing here is applied with raw `kubectl`. Repo seeding is `gh-rest-3`
(repo-create + contents-put, approval-gated); ArgoCD + Application CRs land via
`k8s.apply`; Grafana is viewed in-browser (accepted deviation — no Grafana
connector). See the staging bundle's `ops-sequence.md`.

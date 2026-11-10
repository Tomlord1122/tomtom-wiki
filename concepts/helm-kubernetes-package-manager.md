# Helm: Kubernetes Package Management

Helm packages Kubernetes deployment logic so teams stop copying near-identical YAML per service and environment. Its core equation is:

> **Chart (templates) + Values (parameters) → rendered Kubernetes YAML → Release (installed instance)**

It addresses two scaling failures of hand-managed manifests: **YAML explosion** (dozens of nearly identical Deployment, Service, Ingress, ConfigMap, HPA files across dev/staging/prod) and **configuration drift** (a label, resource limit, or security fix changed in 149 of 150 places).

## The three core nouns

- **Chart** — the versioned package/template blueprint: the recipe describing all resources needed for an application.
- **Values** — the parameter set: ingredients/configuration that fills Chart slots. One Chart can install dev with 1 replica and 128Mi, then prod with 3 replicas, 1Gi, and an enabled Ingress.
- **Release** — a named, installed instance of a Chart in a cluster. Helm tracks its revisions, so it becomes an operational unit rather than a pile of unrelated resources.

A Chart version (`version: 1.2.0`) describes the packaging/template evolution; `appVersion: "3.0.0"` documents the application version it deploys. They intentionally evolve independently.

## Chart structure

`helm create my-chart` produces a common language for reading any Chart:

```text
my-chart/
├── Chart.yaml          # metadata: name, version, description, type
├── values.yaml         # safe-to-override defaults
├── templates/          # Go-templated Kubernetes manifests
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── _helpers.tpl    # reusable named templates; not rendered as a resource
│   └── NOTES.txt       # rendered instructions after install/upgrade
├── charts/             # downloaded/vendored child Chart dependencies
└── .helmignore         # files excluded from the package
```

`templates/` separates structure from environment-specific data. A static `replicas: 3` becomes `replicas: {{ .Values.replicaCount }}`; names often use `{{ .Release.Name }}`. `_helpers.tpl` centralises repeated label/name snippets, so consistency is designed in rather than searched-and-replaced later.

`NOTES.txt` is not deployed to Kubernetes. Helm renders it after an install or upgrade for human operators: a URL, port-forward command, credentials retrieval, or verification step. Treat it as part of the package's UX, not an afterthought.

## Safe CLI workflow

Helm operates on packages and Releases, while `kubectl apply` operates directly on individual resources.

```bash
# inspect/render locally first — no cluster mutation
helm template my-api ./charts/web-app -f values-prod.yaml
helm lint ./charts/web-app

# install once
helm install my-api ./charts/web-app -f values-prod.yaml --namespace production

# inspect deployed state
helm list --namespace production
helm status my-api --namespace production
helm history my-api --namespace production

# change a version or value
helm upgrade my-api ./charts/web-app -f values-prod.yaml --set image.tag=v1.2.0

# recover a known-good revision
helm rollback my-api 1 --namespace production
```

Every install/upgrade becomes a numbered Revision. This gives release history and rollback rather than a frantic search for “the old YAML.” Prefer **render/lint/diff before deploy**, then use explicit values files for reviewed environment changes. Use flags such as `--atomic` when a failed deployment should clean itself up rather than leave a half-applied release.

## Chart sources and dependencies

Charts can come from:

- **HTTP Chart repositories** — index + `.tgz` packages; manage with `helm repo add`, `helm repo update`, `helm search repo`, `helm pull`.
- **OCI registries** — modern registry-based distribution, often beside container images.
- **Local paths** — ideal for your own in-repository Charts during development.

Dependencies make an application package composable: a service may declare Redis or PostgreSQL child Charts in `Chart.yaml`; `helm dependency update` resolves them into `charts/`. Keep dependency versions explicit and review their values/security posture. Convenient packaging does not remove supply-chain responsibility.

## Helmfile and production delivery controls

Helm manages a release. **Helmfile** manages the desired collection of Helm releases across environments. A typical layout uses a shared `helmfile.yaml`, environment-specific values, and `.gotmpl` only where dynamic rendering is truly needed. Keep the rendered values and release versions inspectable, because indirection is useful right up until it becomes a magician's cape.

The newer course notes make the layering explicit:

- Shared defaults live in one place; environment overlays (`int` / `stg` / `prod`) override only the differences.
- Naming conventions stay stable across layers so selectors, ServiceAccounts, and IRSA annotations do not drift by typo.
- Terraform outputs become Helmfile inputs. The app repo does not invent VPC IDs, role ARNs, or cluster names.
- A dual pipeline follows: application CI builds and pushes an image; the Helmfile repo bumps the tag and applies. Image build and cluster apply have different reviewers and different blast radii.

Production delivery needs more than a successful render:

- Pin chart and dependency versions, then plan upgrades as explicit changes.
- Validate value schemas and lint/render before applying.
- Use hooks sparingly for ordered setup or migration work, and make them observable and idempotent.
- Use rolling-update settings and PodDisruptionBudgets to bound availability risk during change.
- Store encrypted secret material with SOPS or fetch it through External Secrets / Secrets Store CSI. Base64 is encoding, not a trench coat for secrets.
- Detect drift between the declared Helmfile state and the live cluster, then reconcile it deliberately.

## Operational framing

Helm is a rendering and release-management layer, not a substitute for understanding Kubernetes. It makes complex resources easier to repeat, version, inspect, and roll back; it does not make insecure manifests or incorrect [[kubernetes-networking]] policies safe. It is especially valuable when infrastructure spans environments: keep the Chart reusable, keep values explicit, and make the rendered result reviewable.

## Related pages

- [[kubernetes-networking]] — Services, Ingress, TLS Secrets, and NetworkPolicy are common Chart-managed resources.
- [[computer-networking-tcp-ip]] — network foundations beneath those Kubernetes resources.
- [[terraform-infrastructure-as-code]] — the infra-repo that publishes identities Helmfile consumes.
- [[eks-platform-engineering]] — IRSA annotations, CSI SecretProviderClass, HPA, and Ingress/ALB releases live here.
- [[cloud-native-ai-infrastructure]] — Helm is part of the operational substrate for platform workloads.

## References

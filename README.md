# AI Runway Takeoff

GitOps manifests for deploying [AI Runway](https://github.com/ai-runway/airunway) and its dependencies to a Kubernetes cluster via [Argo CD](https://argo-cd.readthedocs.io/).

AI Runway is a control plane that unifies multiple inference backends (Kaito, KubeRay, NVIDIA Dynamo, llm-d) behind a single Gateway API surface with the [Gateway API Inference Extension](https://github.com/kubernetes-sigs/gateway-api-inference-extension).

## Repository layout

```
bootstrap.yaml    # ApplicationSet bootstrap (recommended; one repoURL to edit)
app-of-apps.yaml  # Classic app-of-apps Application (alternative entry point)
argocd/apps/      # Individual Argo CD Application manifests
airunway/         # AI Runway controller, gateway, and inference providers
lustre/           # Azure Lustre CSI driver manifests
```

| Argo CD app | Source | Purpose |
|---|---|---|
| `airunway-controller` | `airunway/controller` | AI Runway controller + webhook |
| `airunway-gateway` | `airunway/gateway` | Inference Gateway resources |
| `airunway-providers` | `airunway/providers` | Kaito, KubeRay, Dynamo, llm-d providers |
| `gateway` | upstream | Gateway API + Inference Extension CRDs, Istio |
| `kaito` | upstream Helm | Kaito workspace controller |
| `kuberay` | upstream Helm | KubeRay operator |
| `dynamo` | upstream Helm | NVIDIA Dynamo platform |
| `prometheus` | upstream Helm | Metrics stack |
| `lustre` | `lustre/` | Azure Lustre CSI driver |

## Usage

Pick **one** entry point (don't apply both — they manage the same set of Applications):

**Option A — ApplicationSet (recommended, fork-friendly):**

```sh
kubectl apply -f bootstrap.yaml
```

A single `ApplicationSet` generates the in-repo Apps (`airunway-controller`, `airunway-gateway`, `airunway-providers`, `lustre`) and a `dependencies` App that syncs the upstream charts under `argocd/apps/`.

**Option B — Classic app-of-apps:**

```sh
kubectl apply -f app-of-apps.yaml
```

Points at `argocd/apps/`, which Argo CD expands into all child Apps individually.

## Updating image tags

Provider and controller image tags are pinned in the `kustomization.yaml` files under `airunway/`. Bump the `newTag` value and commit to trigger a sync.

## Forking

**With `bootstrap.yaml` (Option A):** edit the single `repoURL` line in `bootstrap.yaml` and you're done. All generated Applications inherit it.

**With `app-of-apps.yaml` (Option B):** five manifests hardcode this repo's URL (`app-of-apps.yaml` plus the four `airunway-*` / `lustre` files under `argocd/apps/`). Rewrite them in one shot:

```sh
git grep -l pauldotyu/ai-runway-takeoff | xargs sed -i '' 's|pauldotyu/ai-runway-takeoff|YOUR_ORG/YOUR_REPO|g'
```

(Drop the `''` after `-i` on GNU `sed`.) Upstream Helm/Git sources don't need to change.


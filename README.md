# AI Runway Takeoff

GitOps manifests for deploying [AI Runway](https://github.com/kaito-project/airunway) and its dependencies to a Kubernetes cluster via [Argo CD](https://argo-cd.readthedocs.io/).

AI Runway is a control plane that unifies multiple inference backends (Kaito, KubeRay, NVIDIA Dynamo, llm-d) behind a single Gateway API surface with the [Gateway API Inference Extension](https://github.com/kubernetes-sigs/gateway-api-inference-extension).

## Repository layout

```
app-of-apps.yaml  # Root Argo CD Application (app-of-apps pattern)
argocd/apps/      # Argo CD Application manifests (entry point)
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

Bootstrap by applying the root `app-of-apps` Application to a cluster that already has Argo CD installed:

```sh
kubectl apply -f app-of-apps.yaml
```

This single Application points at `argocd/apps/`, which Argo CD then expands into all child apps. Each app syncs from this repo (or its upstream chart) and reconciles changes automatically.

## Updating image tags

Provider and controller image tags are pinned in the `kustomization.yaml` files under `airunway/`. Bump the `newTag` value and commit to trigger a sync.

## Forking

Five manifests hardcode this repo's URL (`app-of-apps.yaml` and the four `airunway-*` / `lustre` Applications under `argocd/apps/`). After forking, repoint them at your fork:

```sh
git grep -l pauldotyu/ai-runway-takeoff | xargs sed -i '' 's|pauldotyu/ai-runway-takeoff|YOUR_ORG/YOUR_REPO|g'
```

(Drop the `''` after `-i` on GNU `sed`.) Upstream Helm/Git sources in the other apps don't need to change.


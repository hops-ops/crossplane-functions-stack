# crossplane-functions-stack

Installs Crossplane **Function** packages — plus a `DeploymentRuntimeConfig` for each — onto a target Crossplane control plane, via `provider-kubernetes`. Split out of the monolithic `CrossplaneStack` so function installation is owned independently of the Crossplane core install, mirroring the per-provider stacks.

Each entry in `spec.functions` renders:
- a `pkg.crossplane.io/v1` `Function` (with `runtimeConfigRef` → its DRC), and
- a `pkg.crossplane.io/v1beta1` `DeploymentRuntimeConfig` carrying resource requests/limits and optional NodePool scheduling.

The `Function` and `DeploymentRuntimeConfig` names are derived from the package path (`<org>-<name>`, e.g. `crossplane-contrib-function-auto-ready`) so compositions can reference them by their canonical name.

## Getting Started

Install the default function set (`function-auto-ready`) onto a cluster:

```yaml
apiVersion: crossplane.ops.com.ai/v1alpha1
kind: FunctionsStack
metadata:
  name: functions
  namespace: default
spec:
  clusterName: my-cluster
  kubernetesProviderConfigRef:
    name: my-cluster
```

Omitting `spec.functions` installs `function-auto-ready:v0.6.0` with default requests (15m CPU / 100Mi memory) — the behavior previously baked into `CrossplaneStack`, now with a DRC.

## Growing

Install several functions, each sized independently, and schedule their runtimes onto the Crossplane NodePool:

```yaml
spec:
  clusterName: my-cluster
  kubernetesProviderConfigRef:
    name: my-cluster
  nodePool:
    enabled: true
  functions:
    autoReady:
      package: xpkg.crossplane.io/crossplane-contrib/function-auto-ready
      version: v0.6.0
    patchAndTransform:
      package: xpkg.crossplane.io/crossplane-contrib/function-patch-and-transform
      version: v0.9.0
      runtimeConfig:
        requestsCpu: 50m
        requestsMemory: 256Mi
        cpu: "1"
        memory: 512Mi
```

Set `enabled: false` on a function entry to stop installing it without removing the declaration.

## Enterprise Scale

- `nodePool.enabled` pins every function runtime to the dedicated `hops-crossplane` Karpenter NodePool (provisioned by `CrossplaneStack`); `scheduling.{nodeSelector,tolerations}` overrides the defaults.
- Per-function `runtimeConfig` sets requests (defaulted to 15m / 100Mi) and optional limits so functions bin-pack predictably.

## Relationship to other stacks

- `CrossplaneStack` installs Crossplane core only (no functions, no providers).
- `crossplane-<x>-provider` stacks install provider packages + ProviderConfigs.
- `crossplane-functions-stack` installs Function packages + their DRCs.

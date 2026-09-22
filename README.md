# k8s-ns-quota-calculator

> This tool was made with the help of AI

[Check it out on github pages!](https://camphul.github.io/k8s-ns-quota-calculator/)

Calculator to help configure Kubernetes namespace resource quotas

A single-file, offline web tool for calculating Kubernetes namespace `ResourceQuota` values from a set of workloads. No server, no dependencies — just open `index.html` in a browser.

## Features

- Define workloads with container resource requests/limits, replica counts, HPA settings, and rolling-update surge
- Attach shared sidecar containers (Istio Proxy, CloudSQL Proxy, etc.) to any workload
- Support for Deployments, StatefulSets, DaemonSets, Jobs, and custom workload types
- Live quota summary (`requests.cpu`, `limits.cpu`, `requests.memory`, `limits.memory`) with optional buffer percentage
- Per-workload and namespace-total breakdown
- Import/export config as JSON; config is automatically saved to `localStorage`

## How to use

1. Clone or download the repo and open `index.html` in any modern browser — no server needed.
2. **Set a buffer %** (default 10%) in the Global Settings row to add headroom on top of calculated totals.
3. **Define sidecars** in the Sidecar Definitions panel. Three common ones are pre-populated; add, edit, or delete as needed.
4. **Add workloads** with the *+ Add Workload* button. For each workload configure:
   - Name and type (Deployment, StatefulSet, DaemonSet, Job, or a custom type)
   - Replica count, or enable HPA and set min/max replicas
   - Max Surge (Deployments only) — extra pods during a rolling update
   - Container CPU and memory requests/limits (in mCPU and MiB)
   - Which sidecars to include (checkboxes under each workload card)
5. The **Namespace Quota** panel on the right updates live. Expand the breakdown to see per-workload contributions for each quota field.
6. Use the **JSON Config** tab to view, edit, or paste a full config. Click *Apply* (or press `Ctrl+Enter`) to load it. Use *Download JSON* to save a copy. Config is also persisted automatically in `localStorage` and restored on next open.

## Calculation

```
effectiveReplicas = (hpa.enabled ? hpa.maxReplicas : replicas) + (surgeCapable ? maxSurge : 0)
perPod            = container + sum(selected sidecars)
workload total    = effectiveReplicas × perPod
namespace quota   = sum(all workloads) × (1 + bufferPercent / 100)
```

Output units: CPU in cores (mCPU ÷ 1000), memory in Gi (MiB ÷ 1024).
# Tunnel Interface Helm Chart

DaemonSet that enables `/dev/net/tun` for VPN and tunnel workloads (e.g. WireGuard) using [squat/generic-device-plugin](https://github.com/squat/generic-device-plugin). Deploy to `kube-system` (or your chosen namespace).

## Subcharts

| Subchart | Source | Values prefix | Description |
|----------|--------|---------------|-------------|
| **onepassworditem** | [expectedbehaviors/OnePasswordItem-helm](https://github.com/expectedbehaviors/OnePasswordItem-helm) | `onepassworditem.*` | Optional secrets sync into the release namespace (e.g. for workloads using the tun device). |

All inputs: **`name`** (DaemonSet/container name; default `generic-device-plugin`), **`onepassworditem.enabled`**, **`onepassworditem.items`**. Defaults: see `values.yaml`.

## Configuration reference (all inputs)

Every value accepted by this chart is documented below. This chart provides a DaemonSet (generic-device-plugin for `/dev/net/tun`) and optionally the onepassworditem subchart.

### Root: chart-specific

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `name` | string | `"generic-device-plugin"` | DaemonSet and container name. |

### Subchart: onepassworditem

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `onepassworditem.enabled` | bool | `true` | Create OnePasswordItem resources; set `false` if not using 1Password. |
| `onepassworditem.defaultVault` | string | `""` | Default vault for items. |
| `onepassworditem.items` | list | `[]` | List of `{ item, name, type }`; optional `vault`, `namespace`, `annotations`, `labels`. |

## Chart contents

- **DaemonSet:** One pod per node; exposes `/dev/net/tun` to the device plugin API.
- **No ingress or secrets** required for the device plugin itself. Runs with `system-node-critical` priority and tolerations so it can run on all nodes.
- **Optional:** Use onepassworditem if you need secrets for workloads that consume the tun device.

## Prerequisites

- **1Password Connect** (optional): if you use `onepassworditem.enabled: true` to sync secrets (e.g. for workloads that consume the tun device). The chart depends on [expectedbehaviors/OnePasswordItem-helm](https://github.com/expectedbehaviors/OnePasswordItem-helm). Set `onepassworditem.enabled: false` if you do not need 1Password.

## Requirements

| Dependency | Notes |
|------------|--------|
| **Namespace** | e.g. `kube-system` (per Argo CD or install target). |
| **Privileged** | DaemonSet uses privileged container and hostPath volumes; nodes must have `/dev/net/tun` available. |

## Values

| Key | Description |
|-----|-------------|
| `name` | DaemonSet and container name (default: `generic-device-plugin`). |
| `onepassworditem.enabled` | If `true` (default), the subchart is enabled. Set `false` if you do not use 1Password. |
| `onepassworditem.items` | List of `{ item, name, type }` for secrets to sync into the release namespace. |

## Install

**From this repo:**

```bash
helm dependency update
helm install tunnel-interface . -f values.yaml -n kube-system
```

**From Helm repo (expectedbehaviors):**

```bash
helm repo add expectedbehaviors https://expectedbehaviors.github.io/tunnel-interface
helm install tunnel-interface expectedbehaviors/tunnel-interface -n kube-system
```

## Render & validation

```bash
helm template tunnel-interface . -f values.yaml -n kube-system
```

## Argo CD

Application in project `kube-system` (or your target namespace), path `.`.

## Next steps

1. Deploy to kube-system; ensure nodes have `/dev/net/tun` available.
2. Workloads that need the tun device can request the corresponding device plugin resource once the DaemonSet is running.

## Template filenames

This chart uses **daemonSets.yaml** (camelCase plural) for the DaemonSet resource.

# Tunnel Interface Helm Chart

DaemonSet that enables `/dev/net/tun` for VPN and tunnel workloads (e.g. WireGuard) using [squat/generic-device-plugin](https://github.com/squat/generic-device-plugin). Deploy to `kube-system` (or your chosen namespace).

## Install

```bash
helm install tunnel-interface . -n kube-system
```

# Host Metrics Collection

## Table of Contents

- [Documentation Reference](#documentation-reference)
- [Host Metrics Overview](#host-metrics-overview)
- [Metrics Categories](#metrics-categories)
  - [CPU Metrics](#cpu-metrics)
  - [Memory Metrics](#memory-metrics)
  - [Load Metrics](#load-metrics)
  - [Disk Metrics](#disk-metrics)
  - [Filesystem Metrics](#filesystem-metrics)
  - [Network Metrics](#network-metrics)
- [Collection Details](#collection-details)

---

## Documentation Reference

This implementation extends the standard Honeycomb Kubernetes OpenTelemetry setup with additional host-level metrics. For complete information about all collected data (including Kubernetes metrics, events, and traces), see the official [Honeycomb documentation on collected data](https://docs.honeycomb.io/send-data/kubernetes/opentelemetry/create-telemetry-pipeline/#collected-data).

## Host Metrics Overview

The DaemonSet collector includes a hostmetrics receiver that gathers comprehensive system metrics from each Kubernetes node. These metrics are collected every 10 seconds and sent to the `k8s-hosts-metrics` dataset in Honeycomb.

## Metrics Categories

### CPU Metrics
- `system.cpu.utilization` (enabled) - CPU utilization percentage per core
- `system.cpu.time` - CPU time spent in different states (user, system, idle, etc.)

### Memory Metrics  
- `system.memory.utilization` (enabled) - Memory utilization percentage
- `system.memory.usage` - Memory usage in bytes (used, free, available, etc.)

### Load Metrics
- `system.cpu.load_average.1m` - 1-minute load average
- `system.cpu.load_average.5m` - 5-minute load average  
- `system.cpu.load_average.15m` - 15-minute load average

### Disk Metrics
- `system.disk.io` - Disk I/O operations (reads/writes)
- `system.disk.io_time` - Time spent on disk I/O
- `system.disk.operation_time` - Time per disk operation

### Filesystem Metrics
- `system.filesystem.usage` - Filesystem usage in bytes (used/free)
- `system.filesystem.utilization` - Filesystem utilization percentage
- `system.filesystem.inodes.usage` - Inode usage

### Network Metrics
- `system.network.io` - Network I/O (bytes/packets sent/received)
- `system.network.errors` - Network errors
- `system.network.dropped` - Dropped network packets
- `system.network.connections` - Network connection counts

## Collection Details
- **Collection Interval**: 10 seconds
- **Deployment**: DaemonSet (one collector per node)
- **Dataset**: `k8s-hosts-metrics` in Honeycomb
- **Access Requirements**: Requires root privileges and host filesystem access via `/hostfs` mount
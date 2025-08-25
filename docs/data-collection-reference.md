# Data Collection Reference

This document provides a comprehensive reference of all telemetry data collected by the Kubernetes Helm Charts in the observability architecture.

## Table of Contents

- [Node & Pod Metrics](#node--pod-metrics)
- [Kubernetes Events](#kubernetes-events)
- [Kubernetes Metadata](#kubernetes-metadata)
- [Application Telemetry (Optional)](#application-telemetry-optional)
- [Host Metrics (Optional)](#host-metrics-optional)
- [Logs (Optional)](#logs-optional)

---

## Node & Pod Metrics

(From **Kubeletstats Receiver** & **Cluster Receiver**)

- **CPU/Memory**: Usage, utilization vs. requests/limits
- **Uptime**: Pod & node uptime
- **Network/Filesystem**: I/O rates, errors, disk usage
- **Counts**: Pods per namespace/node/deployment
- **Restarts**: Container restart counts
- **Cluster State**: Node conditions, pod phases, deployment/statefulset/daemonset/job statuses

## Kubernetes Events

(From **Kubernetes Objects Receiver** \+ **Transform Processor**)

- Pod failures, image pulls, node health, volumes, scheduling issues
- Includes `reason`, `severity`, `namespace`, `pod.name`, `node.name`, etc. (`k8s.*` fields)

## Kubernetes Metadata

(Via **Kubernetes Attributes Processor**)

- Adds labels like: `k8s.pod.name`, `k8s.namespace.name`, `k8s.deployment.name`, `k8s.node.name`, etc.
- Enables correlation across metrics, traces, and logs

## Application Telemetry (Optional)

- Add traces/metrics/logs via OTel SDKs or auto-instrumentation
- Correlate with k8s context (pod, deployment, etc.)
- Useful for latency, error tracking, and business metrics

## Host Metrics (Optional)

- If you add a host metrics receiver to the nodes (DaemonSet), it will collect system-level metrics from the underlying host machines, such as CPU usage, memory utilization, disk I/O, network stats, and filesystem metrics.
- This gives you deeper visibility into the actual node hardware performance beyond what kubeletstats provides from the Kubernetes perspective.

## Logs (Optional)

(Via **Filelog Receiver** or Fluent Bit)

- Collect stdout/stderr, system logs, enriched with `k8s.*`
- Forward to observability backends for centralized log management
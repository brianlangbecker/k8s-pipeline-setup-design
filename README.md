# K8S Pipeline Setup and Design (EARLY DRAFT)

Complete Kubernetes observability solution that extends Honeycomb's OpenTelemetry collector setup with comprehensive host-level system metrics. This configuration provides deep visibility into your K8s clusters by collecting application traces, Kubernetes metrics and events, plus detailed host performance data from CPU, memory, disk, and network resources.

## Table of Contents

- [Architecture Overview](#architecture-overview)
- [Project Structure](#project-structure)
- [Key Setup Sections](#key-setup-sections)
- [Quick Start Guide](#quick-start-guide)
  - [1. Setting Up BindPlane](#setting-up-bindplane-honeycomb-telemetry-pipeline)
  - [2. Deploying OpenTelemetry Collectors](#deploying-opentelemetry-collectors)
  - [3. Verification and Troubleshooting](#troubleshooting)
- [Configuration Files](#configuration-files)
- [Documentation](#documentation)

## Quick Start Guide

Follow these steps in order to deploy the complete observability stack:

1. **[Set up BindPlane](#setting-up-bindplane-honeycomb-telemetry-pipeline)** - Install and configure the telemetry gateway
2. **[Deploy Collectors](#deploying-opentelemetry-collectors)** - Install enhanced OpenTelemetry collectors  
3. **[Verify Data Flow](#troubleshooting)** - Ensure telemetry is flowing correctly

---

## Architecture Overview

This project provides the [Honeycomb Kubernetes OpenTelemetry telemetry pipeline](https://docs.honeycomb.io/send-data/kubernetes/opentelemetry/create-telemetry-pipeline/) configurations enhanced with host metrics collection and routed through BindPlane for centralized telemetry management.

For detailed architecture diagrams and design decisions, see [K8S Architecture.md](K8S%20Architecture.md).

- **Standard Honeycomb Collectors**: Uses official collector configurations for Kubernetes metrics, events, and traces
- **Enhanced Host Metrics**: Adds hostmetrics receiver to DaemonSet collector for comprehensive system-level monitoring
- **BindPlane Gateway**: Routes all telemetry through BindPlane server for centralized processing, transformation, and API key management
- **Multiple Datasets**: Organizes data into `k8s-metrics`, `k8s-events`, `k8s-logs`, and `k8s-hosts-metrics` datasets in Honeycomb

## Project Structure

```
k8s-pipeline-setup-design/
├── configs/           # OpenTelemetry collector configurations
│   ├── values-daemonset.yaml    # Enhanced with hostmetrics receiver
│   └── values-deployment.yaml   # Standard Honeycomb deployment config
├── docs/             # Documentation
│   ├── data-collection-reference.md  # Comprehensive telemetry data details
│   ├── host-metrics.md              # Host metrics collection details
│   └── upgrading-collectors.md      # Upgrade procedures
├── images/           # Architecture diagrams
│   ├── BigPipeLine.png
│   ├── Bindplane.png
│   └── HelmK8S1.png
├── K8S Architecture.md    # Detailed architecture documentation
└── README.md             # This file
```

## Key Setup Sections

### 1. BindPlane Server Setup

BindPlane must be configured first as it provides the gateway endpoint for all collectors:

- **Gateway Collectors**: Centralized telemetry processing and routing
- **API Key Management**: Handles downstream authentication to Honeycomb
- **Data Transformation**: Applies filtering and transformations before forwarding
- **Endpoint Configuration**: Provides the target endpoint for collector configurations

### 2. Enhanced OpenTelemetry Collectors

Once BindPlane is running, deploy the collectors configured to route through it:

- **Deployment-mode Collector**: Collects cluster metrics and Kubernetes events
- **DaemonSet-mode Collector**: Collects node and pod metrics from kubelet plus host metrics

### 3. Host Metrics Enhancement

The DaemonSet collector includes additional system monitoring:

- `hostmetrics` receiver for comprehensive system metrics (CPU, memory, disk, network)
- Proper volume mounts for accessing host filesystem
- Security context for privileged access to host metrics
- Separate dataset (`k8s-hosts-metrics`) for host-level data

### 4. Data Flow

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           Kubernetes Cluster                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐        │
│  │     Node 1      │    │     Node 2      │    │ Deployment      │        │
│  │                 │    │                 │    │ Collector       │        │
│  │ ┌─────────────┐ │    │ ┌─────────────┐ │    │                 │        │
│  │ │ DaemonSet   │ │    │ │ DaemonSet   │ │    │ ┌─────────────┐ │        │
│  │ │ Collector   │ │    │ │ Collector   │ │    │ │• Cluster    │ │        │
│  │ │             │ │    │ │             │ │    │ │  metrics    │ │        │
│  │ │• kubelet    │ │    │ │• kubelet    │ │    │ │• K8s events │ │        │
│  │ │• hostmetrics│ │    │ │• hostmetrics│ │    │ │             │ │        │
│  │ └─────┬───────┘ │    │ └─────┬───────┘ │    │ └─────┬───────┘ │        │
│  │                 │    │                 │    │                 │        │
│  │ ┌─────────────┐ │    │ ┌─────────────┐ │    │                 │        │
│  │ │    Pods     │ │    │ │    Pods     │ │    │                 │        │
│  │ │             │ │    │ │             │ │    │                 │        │
│  │ │• app1       │ │    │ │• app2       │ │    │                 │        │
│  │ │• services   │ │    │ │• services   │ │    │                 │        │
│  │ └─────────────┘ │    │ └─────────────┘ │    │                 │        │
│  └───────┼─────────┘    └───────┼─────────┘    └───────┼─────────┘        │
│          │                      │                      │                  │
│          └──────────┬───────────┼──────────────────────┘                  │
│                     │           │                                         │
│                     │           │                                         │
│                     │           │                                         │
│                     │           │                                         │
│                     │           │                                         │
│                     │           │                                         │
│                     │           │                                         │
└─────────────────────┼───────────┼─────────────────────────────────────────┘
                      │           │
                      │           │
       ┌──────────────┘           │
       │            ┌─────────────┘
       │            │
       │            │
       ▼            ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                           BindPlane Server                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                    Gateway Collectors                              │   │
│  │                                                                     │   │
│  │            • Route and process telemetry data                      │   │
│  │            • Apply transformations and filtering                   │   │
│  │            • Manage downstream API keys                            │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                              Honeycomb                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐  ┌─────────┐ │
│  │   k8s-metrics   │  │   k8s-events    │  │    k8s-logs     │  │k8s-hosts│ │
│  │                 │  │                 │  │                 │  │-metrics │ │
│  │ • Pod metrics   │  │ • K8s events    │  │ • Application   │  │         │ │
│  │ • Node metrics  │  │ • Warnings      │  │   logs          │  │• CPU    │ │
│  │ • Cluster       │  │ • State changes │  │ • Container     │  │• Memory │ │
│  │   metrics       │  │                 │  │   logs          │  │• Disk   │ │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘  │• Network│ │
│                                                                 │• Load   │ │
│  ┌─────────────────┐  ┌─────────────────┐                       └─────────┘ │
│  │      app1       │  │      app2       │                                   │
│  │                 │  │                 │                                   │
│  │ • Traces        │  │ • Traces        │                                   │
│  │ • Custom        │  │ • Custom        │                                   │
│  │ • Events        │  │ • Events        │                                   │
│  └─────────────────┘  └─────────────────┘                                   │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Setting Up BindPlane (Honeycomb Telemetry Pipeline)

**This is the foundation of the entire pipeline architecture.** All OpenTelemetry collectors will route through BindPlane for centralized telemetry management.

### Prerequisites

- Helm 3.0+
- Kubernetes 1.24+
- Honeycomb Telemetry Manager license key (contact your Honeycomb account team)
- AWS ALB for load balancing (optional but recommended for production)

### Install BindPlane in Kubernetes

> **Note**: You'll need a Honeycomb Telemetry Manager license key to use BindPlane. If you don't have one, reach out to Honeycomb to get a Telemetry Manager license key.

```bash
# 1. Add Honeycomb Helm repository
helm repo add honeycomb https://honeycombio.github.io/helm-charts

# 2. Set your Honeycomb license key
export LICENSE_KEY="your-honeycomb-license-key"

# 3. Create Kubernetes secret for HTP
kubectl create secret generic hny-secrets \
    --from-literal=license=$LICENSE_KEY \
    --from-literal=sessions_secret=$(uuidgen) \
    --from-literal=username=admin \
    --from-literal=password=admin

# 4. Install HTP server
helm install htp honeycomb/htp

# 5. (Optional) Port forward to access UI for initial configuration
kubectl port-forward svc/htp 3001
```

**Configure HTP Gateway:**

1. Access HTP UI at `localhost:3001` (admin/admin)
2. Configure Honeycomb API keys for downstream routing
3. Set up gateway collectors to receive OTLP data
4. Note the HTP service endpoint for collector configuration (typically `htp.default.svc.cluster.local:4317`)

**Production Setup (Recommended):**

- Deploy AWS ALB in front of HTP service for high availability
- Update collector configs to use ALB endpoint instead of direct service
- Configure TLS termination at the ALB level

## Deploying OpenTelemetry Collectors

Once BindPlane is running, deploy the collectors that will route telemetry through it.

### Step 1: Create Namespace

Create the OpenTelemetry namespace:

```bash
kubectl create namespace opentelemetry
```

### Step 2: Configure Kubernetes with Your Honeycomb API Key

Within your new namespace, create a Kubernetes Secret that contains your Honeycomb API Key. You can find your Honeycomb API Key in your environment in Honeycomb.

```bash
export HONEYCOMB_API_KEY=mykey
kubectl create secret generic honeycomb --from-literal=api-key=$HONEYCOMB_API_KEY --namespace=opentelemetry
```

### Step 3: Update Configuration Files

Edit both configuration files in the `configs/` directory:

**configs/values-daemonset.yaml:**
- Replace `YOUR_CLUSTER_NAME` with your actual cluster name
- Replace `bindplane-gateway-alb.your-domain.com` with your HTP endpoint:
  - **Direct service**: `htp.default.svc.cluster.local:4317`
  - **With ALB**: `your-htp-alb-endpoint.com:4317`

**configs/values-deployment.yaml:**
- Replace `YOUR_CLUSTER_NAME` with your actual cluster name
- Replace `bindplane-gateway-alb.your-domain.com` with your HTP endpoint:
  - **Direct service**: `htp.default.svc.cluster.local:4317`
  - **With ALB**: `your-htp-alb-endpoint.com:4317`

### Step 4: Add Helm Repository

Add the OpenTelemetry Helm repository. For more details, see the [Honeycomb Kubernetes OpenTelemetry guide](https://docs.honeycomb.io/send-data/kubernetes/opentelemetry/create-telemetry-pipeline/):

```bash
helm repo add open-telemetry https://open-telemetry.github.io/opentelemetry-helm-charts
helm repo update
```

### Step 5: Deploy OpenTelemetry Collectors

Deploy using the official Honeycomb configurations:

```bash
# 1. Deploy Deployment-mode Collector (cluster metrics and events)
helm install otel-collector-cluster open-telemetry/opentelemetry-collector \
  --namespace opentelemetry \
  --values configs/values-deployment.yaml

# 2. Deploy DaemonSet-mode Collector (node and pod metrics)
helm install otel-collector open-telemetry/opentelemetry-collector \
  --namespace opentelemetry \
  --values configs/values-daemonset.yaml
```

### Step 6: Verify Data Flow

1. Check that collectors are sending data to HTP:
   ```bash
   kubectl logs -n opentelemetry daemonset/opentelemetry-node-collector
   kubectl logs -n opentelemetry deployment/opentelemetry-cluster-collector
   ```
2. Monitor HTP UI for incoming telemetry data
3. Verify data flow: collectors → HTP → Honeycomb by checking your Honeycomb UI for incoming metrics and traces

## Troubleshooting

If you're only seeing INFO logs and no data flow, try these debugging steps:

### Enable Debug Mode

The collector configs now include debug logging and debug exporters. After deploying, you should see detailed logs:

```bash
# Check node collector logs (DaemonSet)
kubectl logs -n opentelemetry daemonset/opentelemetry-node-collector -f

# Check cluster collector logs (Deployment)
kubectl logs -n opentelemetry deployment/opentelemetry-cluster-collector -f

# Look for debug exporter output showing actual telemetry data
kubectl logs -n opentelemetry daemonset/opentelemetry-node-collector | grep -A5 -B5 "debug"
```

### Test Connectivity

Test if collectors can reach HTP:

```bash
# Get collector pod name
kubectl get pods -n opentelemetry

# Test connectivity from collector pod to HTP
kubectl exec -n opentelemetry <collector-pod-name> -- wget -qO- --timeout=5 http://htp.default.svc.cluster.local:4317 || echo "Connection failed"

# Check if HTP service is accessible
kubectl get svc -n default htp
kubectl describe svc -n default htp
```

### Verify Configuration

Check your endpoint configurations:

```bash
# Verify the HTP service endpoint
kubectl get svc htp -o wide

# Check if ALB is configured correctly (if using ALB)
kubectl get ingress
```

### Look for Common Issues

1. **Wrong endpoint**: Verify `bindplane-gateway-alb.your-domain.com` is replaced with actual HTP endpoint
2. **Network issues**: Check if collectors can resolve/reach HTP service
3. **No metrics generated**: Verify hostmetrics and kubeletstats are actually collecting data
4. **HTP not configured**: Ensure HTP is set up to receive OTLP data on port 4317

### Debug Output Examples

With debug logging enabled, you should see:

- **Successful data collection**: `"msg":"scraped metrics"` from hostmetrics/kubeletstats
- **Successful export**: `"msg":"Stable TelemetrySettings.Logger is deprecated"` and export attempts
- **Connection errors**: `"msg":"Exporting failed"` with connection details
- **Actual telemetry data**: Detailed output from debug exporter showing metrics/logs/traces

## Key Files in This Repository

- **`configs/values-daemonset.yaml`**: Node-level collector (DaemonSet) with hostmetrics + kubeletstats receivers
- **`configs/values-deployment.yaml`**: Cluster-level collector (Deployment) for cluster metrics and K8s events  
- **`K8S Architecture.md`**: Detailed architecture diagrams and design decisions
- **`docs/`**: Comprehensive documentation for setup, configuration, and troubleshooting

## What's Different from Base Honeycomb Setup

1. **Dual Collector Architecture**:
   - **Node collectors** (DaemonSet): hostmetrics + kubeletstats for per-node data
   - **Cluster collector** (Deployment): cluster metrics + K8s events
2. **Gateway Routing**: All telemetry routes through BindPlane gateway collectors instead of directly to backends
3. **Enhanced Metrics**: Comprehensive system and Kubernetes metrics collection
4. **No API Keys**: BindPlane gateway handles all downstream authentication
5. **Load Balancing**: ALB configuration for high availability gateway access

---

## Configuration Files

This project provides enhanced configurations for OpenTelemetry collectors:

- **[configs/values-daemonset.yaml](configs/values-daemonset.yaml)** - DaemonSet collector with hostmetrics receiver for node-level monitoring
- **[configs/values-deployment.yaml](configs/values-deployment.yaml)** - Deployment collector for cluster metrics and Kubernetes events  

All configurations are pre-configured to route telemetry through BindPlane server before forwarding to Honeycomb.

## Documentation

- **[K8S Architecture.md](K8S%20Architecture.md)** - Detailed architecture diagrams and design decisions
- **[Host Metrics Documentation](docs/host-metrics.md)** - Comprehensive host metrics collection details
- **[Data Collection Reference](docs/data-collection-reference.md)** - Complete telemetry data inventory
- **[Upgrading Collectors](docs/upgrading-collectors.md)** - Upgrade procedures for Helm Chart and BindPlane collectors

For the base Honeycomb Kubernetes setup this project extends, see the [Honeycomb Kubernetes OpenTelemetry documentation](https://docs.honeycomb.io/send-data/kubernetes/opentelemetry/create-telemetry-pipeline/).

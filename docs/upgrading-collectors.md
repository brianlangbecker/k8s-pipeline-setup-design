# Upgrading Collectors

This guide covers upgrade procedures for both Helm Chart collectors and BindPlane collectors in your Kubernetes observability stack.

## Table of Contents

- [Upgrading Helm Chart Collectors](#upgrading-helm-chart-collectors)
- [Upgrading BindPlane Collectors](#upgrading-bindplane-collectors)

---

## Upgrading Helm Chart Collectors

1. Update your Helm repository to get the latest OpenTelemetry chart versions.
2. Verify that your existing collectors are currently running in the honeycomb namespace before starting the upgrade.
3. Upgrade the deployment-mode collector first using the same values file from your original installation \- this collector handles cluster-level metrics and Kubernetes events.
4. Upgrade the daemonset-mode collector next using the same values file from your original installation \- this collector runs on each node and handles kubelet metrics and pod-level data.
5. Check that all collector pods have restarted successfully and are running the new version, looking for both the cluster collector pod and the daemonset collector pods.
6. Wait a few minutes, then check your Honeycomb environment to ensure data is still flowing properly to both your metrics and events datasets.

## Upgrading BindPlane Collectors

**Note**: BindPlane collectors can also be upgraded programmatically via the BindPlane CLI or APIs for automation and CI/CD integration.

1. Open BindPlane UI \- Go to your BindPlane web interface
2. Go to Agents page \- Click on "Agents" in the navigation
3. Select agents to upgrade \- Check the boxes next to agents that need updating
4. Click "Upgrade" \- Hit the upgrade button at the top
5. Choose version \- Select the target version from the dropdown
6. Confirm upgrade \- Click "OK" to start the upgrade
7. Wait and verify \- Watch the status change to confirm agents upgraded successfully
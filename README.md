# Tetrate Pre-Check Tool

A Helm chart for deploying a diagnostic pod that scans Kubernetes clusters for existing Istio installations and generates a comprehensive pre-check report.

## Features

- 🔍 Scans for Istio installations across the cluster
- 📊 Generates detailed diagnostic reports
- 🔄 Updatable script via ConfigMap (no image rebuild needed)
- ☁️ Compatible with major cloud Kubernetes services (EKS, GKE, AKS)
- 📝 Easy log retrieval to local machine

## Prerequisites

- Kubernetes cluster (1.20+)
- Helm 3.x
- kubectl configured to access your cluster

## Repository Structure

```
tetrate-pre-check-tool/
├── Chart.yaml
├── values.yaml
├── README.md
└── templates/
    ├── _helpers.tpl
    ├── configmap.yaml
    ├── deployment.yaml
    ├── rbac.yaml
    └── serviceaccount.yaml
```

## Installation

### 1. Clone the repository

```bash
git clone https://github.com/tetratecx/tetrate-pre-check-tool.git
cd tetrate-pre-check-tool
```

### 2. Deploy the Helm chart

```bash
helm install tetrate-pre-check . -n istio-system --create-namespace
```

### 3. Monitor the pod status

```bash
kubectl get pods -n istio-system -l app.kubernetes.io/name=tetrate-pre-check-tool
```

Wait until the pod status shows "Running".

### 4. Download the report

Once the pod is running, download the diagnostic log to your local machine:

```bash
kubectl cp istio-system/$(kubectl get pod -n istio-system -l app.kubernetes.io/name=tetrate-pre-check-tool -o jsonpath='{.items[0].metadata.name}'):/output/tetrate-pre-check-tool.log ./tetrate-pre-check-tool.log
```

Alternatively, if you know the pod name:

```bash
kubectl cp istio-system/tetrate-pre-check-tool:/output/tetrate-pre-check-tool.log ./tetrate-pre-check-tool.log
```

### 5. View the logs

```bash
cat tetrate-pre-check-tool.log
```

Or follow logs in real-time:

```bash
kubectl logs -n istio-system -l app.kubernetes.io/name=tetrate-pre-check-tool -f
```

## Configuration

### Updating the Pre-Check Script

The diagnostic script is stored in a ConfigMap, allowing you to update it without rebuilding any images:

1. Edit the ConfigMap:

```bash
kubectl edit configmap -n istio-system tetrate-pre-check-tool-scripts
```

2. Or update the `templates/configmap.yaml` file and upgrade the Helm release:

```bash
helm upgrade tetrate-pre-check . -n istio-system
```

3. Delete and recreate the pod to run the updated script:

```bash
kubectl delete pod -n istio-system -l app.kubernetes.io/name=tetrate-pre-check-tool
helm upgrade tetrate-pre-check . -n istio-system
```

### Customizing Values

Edit `values.yaml` or provide overrides during installation:

```yaml
# Change namespace (default: istio-system)
namespace: my-namespace

# Change istioctl version
istioctlVersion: "1.23.2"

# Adjust resource limits
resources:
  limits:
    cpu: 1000m
    memory: 1Gi
  requests:
    cpu: 200m
    memory: 256Mi
```

Apply custom values:

```bash
helm install tetrate-pre-check . -n istio-system --set istioctlVersion=1.22.0
```

## What Gets Checked

The pre-check tool scans for:

- ✅ Istio version (local and remote)
- ✅ Control plane components (pods, deployments, services)
- ✅ Istiod revisions and namespace labels
- ✅ Sidecar injection verification
- ✅ Control plane health validation
- ✅ Proxy status and configuration analysis
- ✅ Webhook configurations
- ✅ Gateway deployments
- ✅ Istio CRDs and custom resources
- ✅ Cluster-wide Istio context

## Uninstallation

```bash
helm uninstall tetrate-pre-check -n istio-system
```

To also remove the namespace (if no other resources exist):

```bash
kubectl delete namespace istio-system
```

## Troubleshooting

### Pod fails to start

Check pod events:
```bash
kubectl describe pod -n istio-system -l app.kubernetes.io/name=tetrate-pre-check-tool
```

### Permission errors

Ensure the ServiceAccount has proper RBAC permissions. The chart creates a ClusterRole with read access to necessary resources.

### Cannot download log file

Verify the pod is running:
```bash
kubectl get pods -n istio-system
```

Check if the log file exists:
```bash
kubectl exec -n istio-system <pod-name> -- ls -la /output/
```

### Updating the script doesn't take effect

After updating the ConfigMap, you must delete and recreate the pod:
```bash
kubectl delete pod -n istio-system -l app.kubernetes.io/name=tetrate-pre-check-tool
```

## License

Apache 2.0

## Support

For issues and questions, please open an issue on the [GitHub repository](https://github.com/tetratecx/tetrate-pre-check-tool/issues).
# OpenCost Installation for Lens

This guide provides installation commands for deploying OpenCost with Lens-specific configuration.

## Prerequisites

- Kubernetes cluster running
- Helm 3.x installed
- `kubectl` configured to access your cluster
- Prometheus running in the `lens` namespace

## Installation

### Install OpenCost with Local Chart

```bash
helm install opencost ./charts/opencost \
  -n opencost \
  -f config/opencost.yaml \
  --create-namespace \
  --set prometheus.enabled=false \
  --set opencost.prometheus.internal.enabled=true \
  --set opencost.prometheus.internal.namespaceName=lens \
  --set opencost.prometheus.internal.serviceName=lens-prometheus-server \
  --set opencost.prometheus.internal.port=9090
```

## Verify Installation

Check that the pods are running:

```bash
kubectl get pods -n opencost
```

Check the service:

```bash
kubectl get svc -n opencost
```

## Access OpenCost UI

### Port Forward to Access Locally

```bash
kubectl port-forward -n opencost service/opencost 9090:9090
```

Then access the UI at: http://localhost:9090

### Access OpenCost API

```bash
kubectl port-forward -n opencost service/opencost 9003:9003
```

Then access the API at: http://localhost:9003

## Configuration Details

The installation uses custom configuration from `config/opencost.yaml` which includes:

- **Prometheus**: Connected to Lens Prometheus at `lens-prometheus-server:9090` in the `lens` namespace
- **Custom Pricing**: Enabled with OCI-specific pricing
- **Persistence**: 50Gi storage on OCI Block Volume
- **Data Retention**: 30 days of daily resolution data
- **Custom Image**: Using OCI Registry image `iad.ocir.io/iduyx1qnmway/lens-metric-collector/opencost:latest`

### Resource Allocation

- **CPU Request**: 10m
- **Memory Request**: 55Mi
- **Memory Limit**: 1Gi

## Upgrade

To upgrade the OpenCost installation:

```bash
helm upgrade opencost ./charts/opencost \
  -n opencost \
  -f config/opencost.yaml \
  --set prometheus.enabled=false \
  --set opencost.prometheus.internal.enabled=true \
  --set opencost.prometheus.internal.namespaceName=lens \
  --set opencost.prometheus.internal.serviceName=lens-prometheus-server \
  --set opencost.prometheus.internal.port=9090
```

## Uninstall

To remove OpenCost:

```bash
helm uninstall opencost -n opencost
```

To also delete the namespace:

```bash
kubectl delete namespace opencost
kubectl get namespace opencost -o json | jq '.spec.finalizers = []' | kubectl replace --raw /api/v1/namespaces/opencost/finalize -f -
```
## Additional Resources

- [Lens Forked OpenCost GitHub](https://github.com/oci-ai-incubations/opencost)
- [OpenCost Documentation](https://www.opencost.io/docs/)
- [OpenCost original Helm Chart Repository](https://github.com/opencost/opencost-helm-chart)


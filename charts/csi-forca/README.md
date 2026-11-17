# csi-forca Helm chart

Chart `0.1.0` deploys driver `1.1.1` using the same production resources as
`releases/1.1.1/deploy.yaml`. The chart owns the driver workloads, RBAC,
services, PDB and CSIDriver. It does not create the namespace, credentials
Secret or a StorageClass. Only one release can be installed per cluster because
the driver name and cluster-scoped resource names are fixed.

## Prerequisites

- A supported Kubernetes cluster with Linux/amd64 nodes prepared for Vitiscale
  NVMe access, as described in the installation guide.
- Helm and Kubernetes permissions to create namespaced workloads, cluster RBAC,
  and the CSIDriver object.
- A `csi-forca-system` namespace that permits the privileged node DaemonSet.
- A `vitiscale-credentials` Secret in that namespace. It must contain `endpoint`
  and either `api-token` or both `username` and `password`. Do not put credentials
  in Helm values: Helm stores values in its release Secret.

## Install from a checkout

```bash
kubectl create namespace csi-forca-system
# Create the vitiscale-credentials Secret before installing the chart.
helm install csi-forca ./charts/csi-forca --namespace csi-forca-system --wait
```

Create a StorageClass separately using a sample in `deploy/samples/`, supplying
the Vitiscale target and storage policy appropriate to your environment.

To limit the node DaemonSet to storage-capable nodes, pass a values file:

```yaml
node:
  nodeSelector:
    kubernetes.io/os: linux
    kubernetes.io/arch: amd64
    csi-forca.datagarden.ru/node: "true"
```

The chart exposes image references, the existing Secret name, controller
replicas and resources, node resources and scheduling, and the RDMA I/O queue
limit in `values.yaml`. The source manifest in `files/production.yaml` is copied
from `dist/deploy.yaml`; `make helm-chart-check` verifies it stays in sync.

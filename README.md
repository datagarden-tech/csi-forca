![Kubernetes](https://img.shields.io/badge/Kubernetes-1.28%20%7C%201.29%20%7C%201.31-326ce5)
![CSI](https://img.shields.io/badge/CSI-1.9+-yellow)

# csi-forca

**csi-forca** is a Kubernetes CSI driver for the Vitiscale NVMe/TCP block storage platform by Datagarden.

* **Volume modes:** Filesystem (ext4, xfs) and Raw Block — RWO
* **Attach model:** Skip-Attach (`CSIDriver.spec.attachRequired=false`)
* **Kubernetes:** 1.28 · 1.29 · 1.31
* **CSI spec:** 1.9+

The CSI driver integrates Kubernetes with the Vitiscale storage system and enables dynamic provisioning and management of persistent volumes backed by Vitiscale NVMe/TCP storage.

---

## Quick Start

```bash
# Kubernetes 1.29 — apply the latest release bundle
kubectl apply -f releases/1.0.0-rc1/deploy-1.29.yaml

# Then apply a StorageClass (choose one)
kubectl apply -f deploy/samples/storageclass-fs-rwo.yaml    # ext4 filesystem
kubectl apply -f deploy/samples/storageclass-block-rwo.yaml # raw block
```

See [Installation](#installation) below for all supported Kubernetes versions and install options.

---

## Container images

```
ghcr.io/datagarden-tech/csi-forca-controller:<tag>
ghcr.io/datagarden-tech/csi-forca-node:<tag>
```

See `CHANGELOG.md` for the current release tag and version notes.

---

## Installation

### Option A — Versioned release bundle (recommended)

Each release publishes signed, scanned deployment bundles under `releases/<version>/`.

```bash
VERSION=1.0.0-rc1

# Kubernetes 1.28
kubectl apply -f releases/${VERSION}/deploy-1.28.yaml

# Kubernetes 1.29
kubectl apply -f releases/${VERSION}/deploy-1.29.yaml

# Kubernetes 1.31
kubectl apply -f releases/${VERSION}/deploy-1.31.yaml
```

### Option B — Convenience bundle from dist/

`dist/deploy-*.yaml` contains the same bundles as the latest export and is updated with each repository push. Use this only for evaluation or when a specific versioned release is not required.

```bash
kubectl apply -f dist/deploy-1.29.yaml
```

### Option C — Kustomize overlay

```bash
kubectl apply -k deploy/overlays/k8s-1-29
```

Base manifests are in `deploy/base/`. Version-specific overlays are in `deploy/overlays/k8s-<version>/`.

### StorageClass samples

Sample StorageClass definitions are in `deploy/samples/`:

* `storageclass-fs-rwo.yaml` — filesystem volumes (ext4 default)
* `storageclass-block-rwo.yaml` — raw block volumes

The driver requires a `vitiscale-credentials` Secret in the `csi-forca-system` namespace with the
Vitiscale REST API endpoint and authentication credentials. Refer to the `deploy/overlays/` manifests
for the expected secret structure.

---

## Release artifacts

Each release publishes a versioned artifact directory:

```
releases/<version>/
├── deploy-1.28.yaml          Deployment bundle — Kubernetes 1.28
├── deploy-1.29.yaml          Deployment bundle — Kubernetes 1.29
├── deploy-1.31.yaml          Deployment bundle — Kubernetes 1.31
├── image-digests.txt         sha256 digests for controller and node images
├── sbom-controller.spdx.json SBOM for the controller image (SPDX)
├── sbom-node.spdx.json       SBOM for the node image (SPDX)
├── scan-report.txt           Vulnerability scan report — human-readable (Trivy)
├── scan-report.json          Vulnerability scan report — machine-readable JSON (Trivy)
├── cosign.pub                Public key for Cosign signature verification
├── RC1-VALIDATION.md         Validation summary — tested capabilities, scope, known limitations
└── signatures/               Offline signature bundles (may be empty; signatures are in the OCI registry)
    ├── csi-forca-controller.sig
    └── csi-forca-node.sig
```

Each release directory includes a validation summary (`RC1-VALIDATION.md`) describing what was tested, what is in scope, and known limitations for that release.

**Verify a container image signature:**

```bash
VERSION=1.0.0-rc1
cosign verify \
  --key releases/${VERSION}/cosign.pub \
  ghcr.io/datagarden-tech/csi-forca-controller@<digest>
```

See `releases/<version>/image-digests.txt` for the canonical digest of each image.

---

## License

The Datagarden software referenced by this repository, including container images and driver binaries, is **proprietary software** distributed under a commercial license agreement with Datagarden.

See the `LICENSE` file for details.

---

## Repository model

This repository is maintained as a **public distribution repository** for the `csi-forca` driver and does not currently accept external contributions.

It is automatically generated from the private development repository and must **not** be edited directly.

The public repository contains only installation manifests, release artifacts, and minimal user-facing documentation required to deploy the driver.

All development and internal documentation are maintained in the upstream private source repository.

If you submit a pull request, you agree that the contribution may be used by Datagarden without restriction as part of Datagarden software.


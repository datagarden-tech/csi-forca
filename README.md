![Kubernetes](https://img.shields.io/badge/Kubernetes-1.34%20%7C%201.35%20%7C%201.36-326ce5)
![CSI](https://img.shields.io/badge/CSI-1.9+-yellow)

# csi-forca

**csi-forca** is a Kubernetes CSI driver for the Vitiscale block storage platform by Datagarden.

* **Volume modes:** Filesystem (ext4, xfs) and Raw Block — RWO
* **Attach model:** Skip-Attach (`CSIDriver.spec.attachRequired=false`)
* **1.1.0 Kubernetes matrix:** 1.34 · 1.35 · 1.36; dependency-constrained per ADR-0018
* **Transport:** NVMe/TCP by default; NVMe/RDMA is opt-in
* **CSI spec:** 1.9+

The CSI driver integrates Kubernetes with the Vitiscale storage system and enables dynamic provisioning and management of persistent volumes backed by Vitiscale NVMe/TCP storage.

---

## Quick Start

```bash
# Create the credentials before the driver pods are started
kubectl create namespace csi-forca-system
# Use the Vitiscale base URL; do not append /api.
kubectl -n csi-forca-system create secret generic vitiscale-credentials \
  --from-literal=endpoint="https://vitiscale.example.com" \
  --from-literal=username="YOUR_USERNAME" \
  --from-literal=password="YOUR_PASSWORD"

# Apply the universal 1.1.x release bundle
kubectl apply -f releases/1.1.0/deploy.yaml

# Then apply a StorageClass (choose one)
kubectl apply -f deploy/samples/storageclass-fs-rwo.yaml    # ext4 filesystem
kubectl apply -f deploy/samples/storageclass-block-rwo.yaml # raw block
```

See [Installation](#installation) below for the support policy and install options.

---

## Container images

```
ghcr.io/datagarden-tech/csi-forca-controller:<tag>
ghcr.io/datagarden-tech/csi-forca-node:<tag>
```

See `CHANGELOG.md` for the current release tag and version notes.

---

## Installation

The detailed Russian installation guide is available from the maintainers
on request. The versioned deployment bundle and verification artifacts are
included in each release.

### Option A — Versioned release bundle (recommended)

Each 1.1.x release publishes one deployment bundle under `releases/<version>/`
and signed, scanned container images in GHCR. The same bundle is used throughout
the selected Kubernetes project matrix.

The versioned bundle is the install source after publication. The public
validation summary identifies the exact tested images and supported scope.

```bash
VERSION=1.1.0
kubectl apply -f releases/${VERSION}/deploy.yaml
```

The default policy selects the latest three upstream-supported Kubernetes minors. For 1.1.0,
the project-supported matrix is 1.34–1.36 because the hardware test environment uses Calico
3.32, whose active Kubernetes test matrix stops at 1.36. Kubernetes 1.37 is deferred. Exact
patch versions, upstream lifecycle status, and test depth are published in the release
`VALIDATION.md`. Kubernetes 1.28–1.31 were validated for 1.0.x and are now legacy/best-effort;
their original versioned bundles remain in the corresponding immutable 1.0.x release directory.

### Option B — Convenience bundle from dist/

`dist/deploy.yaml` contains a convenience copy of the current universal bundle and is updated
with each repository export. Use it only for evaluation or when a versioned release is not
required.

```bash
kubectl apply -f dist/deploy.yaml
```

### Option C — Kustomize overlay

```bash
kubectl apply -k deploy/overlays/vitiscale
```

Base manifests are in `deploy/base/`. A version-specific overlay is created only when Kubernetes
API or runtime behavior actually differs across the selected support matrix.

### StorageClass samples

Sample StorageClass definitions are in `deploy/samples/`:

* `storageclass-fs-rwo.yaml` — filesystem volumes (ext4 default)
* `storageclass-block-rwo.yaml` — raw block volumes

The driver requires a `vitiscale-credentials` Secret in the `csi-forca-system` namespace with the
Vitiscale REST API endpoint and authentication credentials. Create it before applying a release
bundle or a Kubernetes-version overlay. The `endpoint` value is the base URL; do not append `/api`,
because the driver adds API paths such as `/api/auth/login`:

```bash
kubectl -n csi-forca-system create secret generic vitiscale-credentials \
  --from-literal=endpoint="https://vitiscale.example.com" \
  --from-literal=username="YOUR_USERNAME" \
  --from-literal=password="YOUR_PASSWORD"
```

Token authentication is also supported by setting `api-token` instead of `username` and `password`.
See `deploy/samples/secret-vitiscale-credentials.example.yaml` for the complete template.

---

## Release artifacts

Each release publishes a versioned artifact directory:

```
releases/<version>/
├── deploy.yaml               Universal production deployment bundle
├── image-digests.txt         sha256 digests for controller and node images
├── sbom-controller.spdx.json SBOM for the controller image (SPDX)
├── sbom-node.spdx.json       SBOM for the node image (SPDX)
├── scan-report.txt           Vulnerability scan report — human-readable (Trivy)
├── scan-report.json          Vulnerability scan report — machine-readable JSON (Trivy)
├── cosign.pub                Public key for Cosign signature verification
├── VALIDATION.md             Validation summary — tested capabilities, scope, known limitations
└── signatures/               Offline signature bundles (may be empty; signatures are in the OCI registry)
    ├── csi-forca-controller.sig
    └── csi-forca-node.sig
```

Each release directory includes a validation summary describing what was tested, scope, and known limitations for that release.

**Verify a container image signature:**

```bash
VERSION=1.1.0
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

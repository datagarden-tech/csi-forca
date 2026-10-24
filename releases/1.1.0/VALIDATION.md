# csi-forca 1.1.0 — Release Validation Summary

## Release

- **Version:** 1.1.0
- **Date:** 2026-10-24
- **Architecture:** linux/amd64

## Tested Kubernetes Versions

| Version | Test depth | Result |
|---------|------------|--------|
| 1.34.12 | Full two-node Vitiscale lifecycle | PASS |
| 1.35.9 | Two-node TCP/RDMA Filesystem/Block runtime smoke | PASS |
| 1.36.5 | Full two-node lifecycle and four applicable CSI-sanity profiles | PASS |

Kubernetes 1.37 is deferred for this release.

## Validated Capabilities

| Capability | Result |
|------------|--------|
| Dynamic provisioning and deletion | PASS |
| Filesystem volumes (ext4, XFS) and raw Block volumes | PASS |
| NVMe/TCP and opt-in NVMe/RDMA | PASS |
| Publish, unpublish, reconnect, migration and cleanup | PASS |
| Read/write integrity and bounded concurrent I/O | PASS |
| Applicable CSI-sanity profiles on Kubernetes 1.36 | PASS |

The driver uses the CSI Skip-Attach model and supports ReadWriteOnce volumes.
Expansion, snapshots, cloning, ReadWriteMany and topology-aware provisioning
are outside the 1.1.0 release scope.

## Images

| Component | Image | Digest |
|-----------|-------|--------|
| Controller | `ghcr.io/datagarden-tech/csi-forca-controller:1.1.0` | `sha256:c756bae9da0f0180d076525260087dbeb7f5817d0d76521e2884b51e217fd3b3` |
| Node | `ghcr.io/datagarden-tech/csi-forca-node:1.1.0` | `sha256:402fdafcc1b90fe03316fd971e5d8397496137c631983068205555351533da64` |

Use the digest references in `image-digests.txt` for reproducible verification.

## Build and Security Validation

- Go 1.26.10 build, unit tests, `go vet` and production manifest contract: **PASS**.
- SPDX SBOMs are included for both images.
- Trivy scan: **0 HIGH / 0 CRITICAL** for each image with the database and
  scan date recorded in `scan-report.txt`.
- Both image signatures verified by digest with `cosign.pub`, including exact
  digest claims and transparency-log inclusion proofs.
- Public release assets and image digests were checked after publication.

## Release Artifacts

`deploy.yaml`, `image-digests.txt`, both SPDX SBOMs, both scan reports and
`cosign.pub` are in this release directory. The detailed installation guide
is available from the maintainers on request.

## Release Blockers

- None.

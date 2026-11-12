# csi-forca 1.1.1 — Release Validation Summary

## Release

- **Version:** 1.1.1
- **Publication date:** 2026-11-12
- **Architecture:** linux/amd64
- **Status:** final-image runtime, registry and signature validation complete

1.1.1 fixes a race during simultaneous unpublish of volumes sharing an NVMe
subsystem on one node. The published 1.1.0 images remain unchanged.

## Tested Kubernetes Versions

| Version | Test depth | Result |
|---|---|---|
| 1.34.13 | Full two-node lifecycle and 200-volume I/O, integrity and cleanup campaign | PASS with an accepted backend teardown observation |
| 1.35.10 | Full two-node lifecycle | PASS |
| 1.36.6 | Full two-node lifecycle, four applicable CSI-sanity profiles and production-bundle smoke | PASS |

Kubernetes 1.34 reached upstream end of life on 2026-10-27; this row records
driver testing and does not imply upstream Kubernetes maintenance. Kubernetes
1.37 is outside this release's tested matrix.

All 200 volumes passed the measured I/O interval, data integrity checks and
cleanup. One backend diagnostic during teardown was reviewed and accepted for
this patch release; it does not establish a clean backend log.

## Validated Capabilities

| Capability | Result |
|---|---|
| Dynamic provisioning and deletion | PASS |
| Filesystem volumes (ext4 and XFS) and raw Block volumes | PASS |
| NVMe/TCP and opt-in NVMe/RDMA | PASS |
| Concurrent consumers, republish, node migration and cleanup | PASS |
| Read/write integrity under bounded concurrent I/O | PASS |
| Applicable CSI-sanity profiles on Kubernetes 1.36 | PASS |

The driver uses the CSI Skip-Attach model and supports ReadWriteOnce volumes.
Expansion, snapshots, cloning, ReadWriteMany and topology-aware provisioning
are outside the 1.1.1 release scope.

## Images

| Component | Image | OCI index digest |
|---|---|---|
| Controller | `ghcr.io/datagarden-tech/csi-forca-controller:1.1.1` | `sha256:40c96f129300365c9f2239f4703146844dfc820dde1f7cfb2922b026675c0da9` |
| Node | `ghcr.io/datagarden-tech/csi-forca-node:1.1.1` | `sha256:101bc2e75412b0f0320bee9ea67d520d576569d9068f42f34b543055c2d6bc9f` |

Both OCI indexes and their Linux/amd64 manifests are readable anonymously.
The exact digests are also recorded in `image-digests.txt`.

## Build and Security Validation

- Go 1.26.10 build, unit tests with race detector, `go vet`, module verification
  and production manifest contract: **PASS**.
- SPDX SBOMs are prepared for both images.
- Trivy rescan on 2026-11-12 with a database refreshed 2026-11-11:
  **0 HIGH / 0 CRITICAL** and zero secret findings for each image.
- Both remote digest references: **Cosign verified** with `cosign.pub`,
  including transparency-log inclusion.

## Release Artifacts

The versioned deployment bundle, image digests, public Cosign key, both SPDX
SBOMs and both scan reports are prepared. The detailed installation guide is
available from the maintainers on request.

## Publication

This summary accompanies the 1.1.1 GitHub Release.

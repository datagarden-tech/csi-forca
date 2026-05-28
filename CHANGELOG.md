# CHANGELOG — csi-forca

This file records notable changes per release. Format: [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

---

## [1.0.0] — 2026-05-28

General Availability release. GA gate cleared.

### Kubernetes runtime matrix

| Version | Overlay / dist bundle | Status |
|---------|----------------------|--------|
| 1.28.15 | `k8s-1-28` / `deploy-1.28.yaml` | ✅ smoke.e2e PASS |
| 1.29.15 | `k8s-1-29` / `deploy-1.29.yaml` | ✅ smoke.e2e PASS |
| 1.30.14 | `test-cluster` overlay (no dedicated dist bundle) | ✅ smoke.e2e PASS |
| 1.31.14 | `k8s-1-31` / `deploy-1.31.yaml` | ✅ smoke.e2e PASS |

Kubernetes 1.30 is runtime-validated using the existing test-cluster overlay; no dedicated 1.30 dist bundle is shipped.

### Changes since RC1

- **grpc CVE-2026-33186 remediated** — `google.golang.org/grpc` upgraded from v1.76.0 to v1.79.3 (CRITICAL, authorization bypass via HTTP/2 path validation).
- `mkfsPolicy=safe` validated end-to-end across all supported Kubernetes versions.
- `allowVolumeExpansion=false` is set in all StorageClass examples; volume expansion is not supported in the GA scope.
- Release artifacts include signed container images (cosign), SBOMs (SPDX), image digests, and clean vulnerability scans (0 HIGH/CRITICAL).
- Validation report renamed from `RC1-VALIDATION.md` to `VALIDATION.md` for all GA and future releases.

### Validation artifacts

See [releases/1.0.0/VALIDATION.md](releases/1.0.0/VALIDATION.md) for the full GA validation record.

---

## [1.0.0-rc1] — 2026-03-08 (published 2026-03-10)

First release candidate. Targets production NVMe/TCP block storage with the Vitiscale backend.

Public GitHub release published: [https://github.com/datagarden-tech/csi-forca/releases/tag/1.0.0-rc1](https://github.com/datagarden-tech/csi-forca/releases/tag/1.0.0-rc1)

### Supported Kubernetes versions

`1.28` · `1.29` · `1.31`

Validated on a multi-node kubeadm cluster (k8s 1.28.15, containerd, 100 GbE networking).

### Canonical identities

| Artifact | Value |
|----------|-------|
| CSI driver name | `csi.forca.datagarden.tech` |
| Go module | `datagarden.tech/csi-forca` |
| Container images | `ghcr.io/datagarden-tech/csi-forca-{controller,node}` |
| Provisioner field | `csi.forca.datagarden.tech` |
| StorageClass param key | `csi.forca.datagarden.tech/mkfsPolicy` |

### Scope for RC1

**Included:**
- Filesystem volumes (ext4, xfs) — RWO
- Raw block volumes — RWO
- NVMe/TCP backend (Vitiscale REST API)
- Native NVMe multipath (round-robin policy recommended)
- Skip-Attach model (`CSIDriver.spec.attachRequired=false`)
- `GET_VOLUME_STATS` / `LIST_VOLUMES` / `GET_CAPACITY` CSI capabilities
- Prometheus metrics (controller + node; NodePort services included)
- Kustomize overlays for k8s-1-28, k8s-1-29, k8s-1-31
- Single-file deployment bundles: `dist/deploy-1.28.yaml`, `dist/deploy-1.29.yaml`, `dist/deploy-1.31.yaml`

**Not included in RC1:**
- Volume snapshots / clones
- Volume expansion
- Topology / zone awareness
- iSCSI or NVMe/RDMA connectors
- Multi-arch images (x86_64 only)
- CI-automated e2e gating

### Major capabilities validated before RC1

| Capability | Validation |
|-----------|------------|
| Filesystem mode (ext4/xfs), 100 concurrent × 900 s | ✅ PASS (2026-02-21) |
| Block volumeMode, 100 concurrent × 900 s | ✅ PASS (2026-02-20) |
| NVMe multipath round-robin — 3.6× write tail latency improvement | ✅ Confirmed (2026-02-21) |
| Fault injection: control-plane restart, CRI restart, kubelet restart | ✅ All PASS (2026-02-22) |
| Fault injection: NVMe portal degradation (1-of-3, 2-of-3) | ✅ All PASS (2026-02-22/23) |
| Fault injection: hard power cycle (IPMI), safe+zeroing | ✅ PASS (2026-02-24) |
| Multi-node: PVC migration, parallel mounts, pod reschedule | ✅ All PASS (2026-03-05) |
| Multi-node: burst provisioning 100 PVC / 100 pods (50/node) | ✅ PASS (2026-03-05) |
| Multi-node: NVMe subsystem reuse, 20 serial + 20 cross-node cycles | ✅ PASS (2026-03-05) |
| NodeGetVolumeStats (filesystem + block paths) | ✅ PASS (2026-03-06) |
| ListVolumes / GetVolume (Vitiscale backend) | ✅ PASS (2026-03-07) |
| Post-identity-migration smoke (fio 5 min randrw) | ✅ PASS (2026-03-08) |

### Important notes for pre-RC adopters

**Breaking: driver identity changed.** No backward compatibility with `dg.csi.forca` is provided.

If you deployed a pre-RC version of this driver, you must:

1. Remove all `PersistentVolume` objects provisioned by `dg.csi.forca` (volumes cannot be
   migrated in-place across driver identity changes).
2. Remove the old `CSIDriver` object (`kubectl delete csidriver dg.csi.forca`).
3. Remove the kubelet plugin directory:
   `rm -rf /var/lib/kubelet/plugins/dg.csi.forca` (on every worker node).
4. Update all `StorageClass` objects: change `provisioner: dg.csi.forca` →
   `provisioner: csi.forca.datagarden.tech` and parameter key
   `csi-forca.datagarden.io/mkfsPolicy` → `csi.forca.datagarden.tech/mkfsPolicy`.
5. Deploy the RC1 driver using the new overlays or dist bundles.

**mkfsPolicy default:** `safe`. Under `safe`, the driver never destroys an existing filesystem.
Use `always` only in scratch/test environments where data loss is acceptable.

**zeroing default:** `true` (Vitiscale). New volumes are zero-initialized before first use,
preventing residual-data reads. Set `vitiscale.zeroing: "false"` only when the application
guarantees full overwrite before any security-sensitive read.

**NVMe multipath policy:** `round-robin` is recommended for production Vitiscale deployments.
It improves write tail latency by 3.6× compared to the kernel default (NUMA).
Configure via udev rule on all worker nodes (see release documentation for details).

### Known limitations (post-RC1 follow-up)

- `PrepareDevice` does not distinguish transient block-0 EIO from absent filesystem under
  `mkfsPolicy=always`. Mitigation: use `mkfsPolicy=safe` (the default) with `zeroing=true`.
  Full EIO retry/backoff is tracked as a follow-up item.
- `ListVolumes` paginates in-memory (Vitiscale returns all volumes in a single response).
  Adequate for expected scale; server-side pagination requires a Vitiscale API enhancement.
- No CI-automated e2e gating. Smoke tests are run manually per release.

---

## [Pre-RC development] — 2026-01 through 2026-03-07

Internal development iterations. Not tagged.

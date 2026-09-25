# Release Validation Summary

## Release

- **Version:** 1.0.1
- **Date:** 2026-09-25

## Tested Kubernetes Versions

| Version | Overlay | Status |
|---------|---------|--------|
| 1.28.0 API / 1.28.15 kubelets | `k8s-1-28` / `test-cluster` | ✅ 1.0.1 smoke.e2e PASS |
| 1.29.15 | `k8s-1-29` | ✅ 1.0.1 manifest PASS; 1.0.0 runtime baseline PASS |
| 1.30.14 | `test-cluster` (no dedicated dist overlay) | ✅ 1.0.0 runtime baseline PASS; not re-run for patch |
| 1.31.14 | `k8s-1-31` | ✅ 1.0.1 manifest PASS; 1.0.0 runtime baseline PASS |

## Validated Capabilities

| Capability | Status | Notes |
|------------|--------|-------|
| Dynamic provisioning (CreateVolume / DeleteVolume) | ✅ PASS | 1 GiB lifecycle validated |
| ControllerPublishVolume / ControllerUnpublishVolume | N/A | Skip-Attach model |
| NodePublishVolume / NodeUnpublishVolume | ✅ PASS | Mount, RW, unmount and cleanup validated |
| Filesystem volumes — ext4 | ✅ PASS | `mkfsPolicy=safe`; write/read verified |
| Filesystem volumes — xfs | ✅ Baseline PASS | 1.0.0 runtime baseline; not re-run for patch |
| Raw block volumes | ✅ Baseline PASS | 1.0.0 runtime baseline; not re-run for patch |
| NodeGetVolumeStats | ✅ Unit + baseline PASS | 1.0.0 runtime baseline; unit revalidated for 1.0.1 |
| NVMe/TCP connect / disconnect | ✅ PASS | 3 of 6 advertised portals reachable; subsystem disconnected cleanly |
| NVMe native multipath | ✅ PASS | Three-path connection established; partial-path handling validated |
| Metrics exporter — controller | ✅ Unit PASS | |
| Metrics exporter — node | ✅ Unit PASS | |

## Scope of This Release

- Single-node access patterns only
- ReadWriteOnce (RWO) access mode
- Vitiscale NVMe/TCP backend

## Not Supported in This Release

- Volume expansion (`ControllerExpandVolume` / `NodeExpandVolume`)
- Snapshots (`CreateSnapshot` / `DeleteSnapshot`)
- Volume cloning
- Multi-node ReadWriteMany (RWX) semantics
- Topology-aware provisioning

## Images

| Component | Image | Digest |
|-----------|-------|--------|
| Controller | `ghcr.io/datagarden-tech/csi-forca-controller:1.0.1` | `sha256:fa9219df85e504a4dd718f7b69352fef1d70e20d1f3a285c1ade95036d073115` |
| Node | `ghcr.io/datagarden-tech/csi-forca-node:1.0.1` | `sha256:a03f87218d6e0c06f46e207fa88141232470d14c9be34d2703bccdb169948241` |

## Canonical Install Source

```
releases/1.0.1/
```

## Verification Artifacts

| Artifact | File | Present |
|----------|------|---------|
| SBOM — controller | `sbom-controller.spdx.json` | ✅ |
| SBOM — node | `sbom-node.spdx.json` | ✅ |
| Vulnerability scan (text) | `scan-report.txt` | ✅ |
| Vulnerability scan (JSON) | `scan-report.json` | ✅ |
| Image digests | `image-digests.txt` | ✅ |
| Cosign public key | `cosign.pub` | ✅ |

## Build and Security Validation

- `go mod verify`, `make build-bins unit manifest-contract`: **PASS**
- Official `govulncheck ./...`: **PASS** — no known vulnerabilities
- Container build-time binary smoke checks: **PASS**
- Container `--version`: **PASS** — version `1.0.1`, revision `543b2da29d9f`
- `google.golang.org/grpc` upgraded to `v1.83.2`; CVE-2026-84445 remediated
- Trivy release gate (database refreshed 2026-09-25): **PASS** — 0 HIGH / 0 CRITICAL for controller and node
- Cosign verification by image digest: **PASS** — claims, offline transparency
  log entries, and signatures validated for controller and node
- Runtime `make smoke.e2e --skip-build`: **PASS** on Kubernetes 1.28.0 (API server)
  with 1.28.15 kubelets using the rebuilt `1.0.1` images. Dynamic provisioning,
  ext4 mount, read/write,
  unpublish, NVMe subsystem disconnect, volume deletion, and cleanup passed.
  Evidence: `_evidence/smoke-e2e-20260925T151807Z/`.

## Patch Release Test Policy

The complete Kubernetes 1.28–1.31 runtime matrix was validated for 1.0.0. For
this 1.0.1 patch, Kubernetes 1.28 (1.28.0 API server, 1.28.15 kubelets)
was selected for representative runtime revalidation. The 1.29 and 1.31 release
manifests were rebuilt and validated by the production manifest contract. Kubernetes 1.30 has no dedicated release
bundle and retains the 1.0.0 runtime baseline. Runtime repetition on every
baseline minor is not a release blocker for this patch.

## Release Blockers

- None.

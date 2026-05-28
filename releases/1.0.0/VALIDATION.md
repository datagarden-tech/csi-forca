# Release Validation Summary

## Release

- **Version:** 1.0.0
- **Date:** 2026-05-28

## Tested Kubernetes Versions

| Version | Overlay | Status |
|---------|---------|--------|
| 1.28 | `k8s-1-28` | ✅ PASS — smoke.e2e |
| 1.29 | `k8s-1-29` | ✅ PASS — smoke.e2e |
| 1.30 | `test-cluster` (runtime-validated; no dedicated dist overlay) | ✅ PASS — smoke.e2e |
| 1.31 | `k8s-1-31` | ✅ PASS — smoke.e2e |

## Validated Capabilities

| Capability | Status | Notes |
|------------|--------|-------|
| Dynamic provisioning (CreateVolume / DeleteVolume) | ✅ PASS | |
| ControllerPublishVolume / ControllerUnpublishVolume | N/A | Skip-Attach model |
| NodePublishVolume / NodeUnpublishVolume | ✅ PASS | |
| Filesystem volumes — ext4 | ✅ PASS | mkfsPolicy=safe validated |
| Filesystem volumes — xfs | ✅ PASS | mkfsPolicy=safe validated |
| Raw block volumes | ✅ PASS | |
| NodeGetVolumeStats | ✅ PASS | |
| NVMe/TCP connect / disconnect | ✅ PASS | |
| NVMe native multipath | ✅ PASS | |
| Metrics exporter — controller | ✅ PASS | |
| Metrics exporter — node | ✅ PASS | |

## Scope of This Release

- Single-node access patterns only
- ReadWriteOnce (RWO) access mode
- Vitiscale NVMe/TCP backend
- `allowVolumeExpansion: false` — volume expansion is not supported in this release

## Not Supported in This Release

- Volume expansion (`ControllerExpandVolume` / `NodeExpandVolume`)
- Snapshots (`CreateSnapshot` / `DeleteSnapshot`)
- Volume cloning
- Multi-node ReadWriteMany (RWX) semantics
- Topology-aware provisioning

## Images

| Component | Image |
|-----------|-------|
| Controller | `ghcr.io/datagarden-tech/csi-forca-controller:1.0.0` |
| Node | `ghcr.io/datagarden-tech/csi-forca-node:1.0.0` |

## Canonical Install Source

```
releases/1.0.0/
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

## Security Scan Result

Vulnerability scan: **0 HIGH / 0 CRITICAL** findings.

Gate: no HIGH/CRITICAL findings allowed — **CLEARED**.

Images signed with cosign. Verify with:

```bash
cosign verify --key releases/1.0.0/cosign.pub ghcr.io/datagarden-tech/csi-forca-controller:1.0.0
cosign verify --key releases/1.0.0/cosign.pub ghcr.io/datagarden-tech/csi-forca-node:1.0.0
```

## CVE Remediation

| CVE | Severity | Package | Fixed in |
|-----|----------|---------|----------|
| CVE-2026-33186 | CRITICAL | `google.golang.org/grpc` v1.76.0 (authorization bypass via HTTP/2 path validation) | v1.79.3 — included in this build |

## Known Limitations

- `ListVolumes` paginates in-memory (Vitiscale returns all volumes in a single response). Adequate for expected scale; server-side pagination requires a Vitiscale API enhancement.
- No CI-automated e2e gating. Smoke tests are run manually per release.
- `PrepareDevice` does not distinguish transient block-0 EIO from absent filesystem under `mkfsPolicy=always`. Mitigation: use `mkfsPolicy=safe` (the default) with `zeroing=true`.

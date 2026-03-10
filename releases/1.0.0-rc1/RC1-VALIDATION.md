# RC1 Validation Summary

## Release

- **Version:** 1.0.0-rc1
- **Date:** 2026-03-09

## Tested Kubernetes Versions

| Version | Overlay | Status |
|---------|---------|--------|
| 1.28 | `k8s-1-28` | |
| 1.29 | `k8s-1-29` | |
| 1.31 | `k8s-1-31` | |

## Validated Capabilities

| Capability | Status | Notes |
|------------|--------|-------|
| Dynamic provisioning (CreateVolume / DeleteVolume) | | |
| ControllerPublishVolume / ControllerUnpublishVolume | N/A | Skip-Attach model |
| NodePublishVolume / NodeUnpublishVolume | | |
| Filesystem volumes — ext4 | | |
| Filesystem volumes — xfs | | |
| Raw block volumes | | |
| NodeGetVolumeStats | | |
| NVMe/TCP connect / disconnect | | |
| NVMe native multipath | | |
| Metrics exporter — controller | | |
| Metrics exporter — node | | |

## Scope of This Release

- Single-node access patterns only
- ReadWriteOnce (RWO) access mode
- Vitiscale NVMe/TCP backend

## Not Supported in RC1

- Volume expansion (`ControllerExpandVolume` / `NodeExpandVolume`)
- Snapshots (`CreateSnapshot` / `DeleteSnapshot`)
- Volume cloning
- Multi-node ReadWriteMany (RWX) semantics
- Topology-aware provisioning

## Images

| Component | Image |
|-----------|-------|
| Controller | `ghcr.io/datagarden-tech/csi-forca-controller:1.0.0-rc1` |
| Node | `ghcr.io/datagarden-tech/csi-forca-node:1.0.0-rc1` |

## Canonical Install Source

```
releases/1.0.0-rc1/
```

## Verification Artifacts

| Artifact | File | Present |
|----------|------|---------|
| SBOM — controller | `sbom-controller.spdx.json` | |
| SBOM — node | `sbom-node.spdx.json` | |
| Vulnerability scan (text) | `scan-report.txt` | |
| Vulnerability scan (JSON) | `scan-report.json` | |
| Image digests | `image-digests.txt` | |
| Cosign public key | `cosign.pub` | |

## Security Notes

### CVE-2026-0861 — glibc memalign integer overflow (HIGH)

**Package:** `libc6` 2.36-9+deb12u13 (Debian bookworm base image)
**Severity:** HIGH
**Status:** affected — no upstream fix available for Debian bookworm at time of RC1 release
**Reference:** https://avd.aquasec.com/nvd/cve-2026-0861

**Risk evaluation:**
The vulnerability requires attacker control over both alignment and size parameters in
`memalign`-family calls. The CSI driver does not expose such inputs in its code path:
all memory allocation is internal and not driven by external (attacker-controlled) input.

**Decision:**
Accepted risk for RC1. The issue will be monitored and resolved once the upstream
distribution publishes a patched `libc6` package for Debian bookworm. A rebuild against
the patched base image will be performed at that time.

## Known Limitations

- (none documented)

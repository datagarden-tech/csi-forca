# Security Policy

## Reporting a Vulnerability

**Do not report security vulnerabilities through public GitHub issues.**

To report a security issue privately, contact the Datagarden by email:

**info@datagarden.tech**

Include a description of the issue, steps to reproduce, and the affected version.
You will receive an acknowledgement within 5 business days.

For general licensing or commercial inquiries, contact: info@datagarden.tech

---

## Supported Releases

| Release | Status |
|---------|--------|
| 1.0.0-rc1 | Supported (current) |

Only the most recent release receives security fixes.
Older releases are not patched.

---

## Image Signing

All release container images are signed with [Cosign](https://github.com/sigstore/cosign).

The public verification key is published with each release:

```
releases/<version>/cosign.pub
```

### Verify a container image

```bash
VERSION=1.0.0-rc1

cosign verify \
  --key releases/${VERSION}/cosign.pub \
  ghcr.io/datagarden-tech/csi-forca-controller:${VERSION}

cosign verify \
  --key releases/${VERSION}/cosign.pub \
  ghcr.io/datagarden-tech/csi-forca-node:${VERSION}
```

See `releases/<version>/image-digests.txt` for the sha256 digest of each image.
Verification by digest is recommended for production deployments.

---

## Vulnerability Scanning

Each release includes a Trivy vulnerability scan report:

```
releases/<version>/scan-report.txt   (human-readable)
releases/<version>/scan-report.json  (machine-readable)
```

Known vulnerabilities accepted at release time are documented in
`releases/<version>/RC1-VALIDATION.md` under the Security Notes section.

---

## Software Bill of Materials

SPDX SBOMs are published for each container image:

```
releases/<version>/sbom-controller.spdx.json
releases/<version>/sbom-node.spdx.json
```

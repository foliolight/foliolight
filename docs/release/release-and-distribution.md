# Release, Signing, Distribution, and Update Plan

Status: Draft v0.1. Several choices here depend on [open questions](../community/open-questions.md); recommended defaults are shown.

## Versioning

- Semantic versioning: `MAJOR.MINOR.PATCH`. Pre-releases use `-beta.N` or `-rc.N`.
- Spec versions (view schema, pack, comments, config) are independent of the app version and documented in each spec.

## Artifacts per release

| Artifact | Purpose |
| --- | --- |
| `foliolight-<ver>-windows-x64.zip` | Portable build for x64 |
| `foliolight-<ver>-windows-arm64.zip` | Portable build for ARM64 (ASSUMPTION A-10) |
| `foliolight-<ver>-windows-x64-webview2-fixed.zip` | Larger build bundling a fixed-version WebView2 runtime, for Windows Server or images without WebView2 |
| `SHA256SUMS` | SHA-256 of every artifact |
| `SHA256SUMS.minisig` | Minisign signature over `SHA256SUMS` |
| `latest.json` and `latest.json.minisig` | Signed update manifest |
| `foliolight-<ver>.cdx.json` | CycloneDX SBOM (Rust and npm dependencies) |
| `THIRD_PARTY_LICENSES.txt` | Generated license inventory (also inside each zip) |
| Build provenance attestation | GitHub artifact attestation for each zip |
| Release notes | Changes, security fixes, spec version changes, known issues |

### Zip contents

```
foliolight-1.2.0-windows-x64/
  foliolight.exe              Authenticode-signed GUI
  foliolight-cli.exe          Authenticode-signed CLI
  README.txt                  Quick start and disclaimer
  LICENSE
  NOTICE
  THIRD_PARTY_LICENSES.txt
  sbom.cdx.json
  portable-mode.txt           How to enable portable mode (rename to foliolight.portable)
```

Portable mode is off by default in the zip, so a copy placed in an approved shared location does not try to write next to itself. Users or admins enable it by creating `foliolight.portable`.

The target is two self-contained executables with no side-by-side DLLs. The Phase 0 spike must confirm that WebView2 loading is statically linked; if a loader DLL is required, it is signed and listed in the hash inventory.

## Signing

### Authenticode (executables)

| Option | Agency acceptance | Cost and effort | Notes |
| --- | --- | --- | --- |
| **Azure Trusted Signing** (recommended) | Good: Microsoft-issued certificates chain to a trusted root and are recognized by SmartScreen | Low monthly cost; integrates with GitHub Actions through OpenID Connect, so no long-lived secret is stored | Requires identity validation of an eligible organization (ASSUMPTION A-03). Short-lived certificates mean publisher identity is stable but thumbprints change; WDAC and AppLocker rules should target publisher and product name, not certificate thumbprint. |
| OV certificate in a cloud HSM (for example, Azure Key Vault) | Good | Moderate yearly cost; key must be in hardware since 2023 baseline requirements | Stable thumbprint. Requires an organization to purchase and hold. |
| EV certificate | Good | Highest cost; hardware token or cloud HSM | No longer provides an instant SmartScreen reputation boost compared with OV (Microsoft changed this in 2024; verify current behavior before purchase). |
| Sigstore only | Low for Windows: Authenticode is what Windows, AppLocker, and WDAC understand | Free | Excellent for provenance; used in addition to Authenticode, not instead of it. |
| Unsigned | Poor: blocked by Smart App Control, warned by SmartScreen, hard to allowlist except by hash | Free | Acceptable only for early test builds. |

**Current tentative decision: executables will probably not be Authenticode-signed.** The baseline release then relies on SHA-256 hashes, the minisign signature over `SHA256SUMS`, and GitHub build provenance attestations. Consequences to document for agency IT:

- SmartScreen shows "Windows protected your PC" on first run until the user selects "More info" then "Run anyway", if policy allows that at all.
- Smart App Control (Windows 11) blocks unsigned apps outright.
- AppLocker and WDAC allowlisting must use **hash rules** (or path rules on an admin-controlled folder), which have to be updated for every release. Publisher rules are not possible.
- Some antivirus products are more likely to flag unsigned executables.

If agencies report that these block adoption, revisit Authenticode signing (Azure Trusted Signing through a neutral organization is the lowest-effort path).

**Who holds the signing identity**, if signing is adopted later, is an open question. Recommended default: the CSP trade organization (or another neutral legal entity) holds it, and signing runs only in the protected release environment, so no individual maintainer holds the key.

### Minisign (update manifest and checksums)

- An Ed25519 minisign key pair signs `SHA256SUMS` and `latest.json`.
- Two public keys are embedded in each binary: the current key and the next key. Rotation: publish a release signed by the current key that announces the next key, then switch.
- Secret keys are stored as protected release-environment secrets (or in a hardware-backed store), never on maintainer laptops.

### Provenance

GitHub artifact attestations record which workflow, commit, and runner produced each artifact. Agencies can verify them with the GitHub CLI (`gh attestation verify`).

## Release process

1. Release pull request: version bump, changelog, updated third-party inventory, spec version notes.
2. Two maintainer approvals; all CI checks green (tests, fuzz smoke, license scan, cargo-deny, accessibility checks).
3. A maintainer creates a **signed tag**.
4. CI builds in a clean runner with pinned toolchains and `--locked` dependencies.
5. The signing job runs in a protected environment that requires two maintainer approvals; it signs executables, produces `SHA256SUMS`, signatures, `latest.json`, SBOM, and attestations.
6. A draft GitHub release is created. A second maintainer downloads it and runs `foliolight-cli update verify` and an attestation check on a clean machine.
7. The release is published. `latest.json` becomes visible to update checks only at this point.

Security fixes: best-effort targets of 14 days for critical and 30 days for high severity, as described in SECURITY.md. As a volunteer stopgap project, these are goals, not guarantees.

## Distribution channels

| Channel | Status | Notes |
| --- | --- | --- |
| GitHub Releases (portable zip) | Primary | Includes hashes, signatures, SBOM, attestations |
| MSI installer (per-user, no admin) | Deferred, post-1.0 | For agencies whose software catalogs require an installer. Would still write no services or auto-start entries. |
| winget manifest | Deferred, community | Useful for some agencies; depends on catalog policy |
| MSIX | Not recommended | Container model complicates arbitrary folder and share access and requires a package identity |
| Agency internal catalogs | Supported by documentation | We publish hashes, SBOM, and an "IT administrator guide" |

## Update check and download-and-verify

### Check

- Off until the user opts in on first run (ASSUMPTION A-15). Disabled entirely by policy for air-gapped environments.
- Request: `GET https://github.com/<org>/foliolight/releases/latest/download/latest.json` (and its `.minisig`). This uses release asset downloads rather than the rate-limited REST API (60 unauthenticated requests per hour per IP, which shared agency egress addresses can exceed).
- Proxy and firewall teams need to allow `github.com` and GitHub's release asset download host. The exact host names must be confirmed in Phase 1 and listed in the administrator guide.
- At most once per launch and once per 24 hours; 5 second timeout; system proxy; operating system certificate store.
- The manifest is verified with the embedded keys before any field is trusted.

`latest.json` example:

```json
{
  "schema": "foliolight/release-manifest/v1",
  "version": "1.3.0",
  "released": "2026-11-10",
  "notesUrl": "https://github.com/foliolight/foliolight/releases/tag/v1.3.0",
  "security": false,
  "artifacts": [
    { "name": "foliolight-1.3.0-windows-x64.zip", "sha256": "…", "size": 8123456 },
    { "name": "foliolight-1.3.0-windows-arm64.zip", "sha256": "…", "size": 7987654 }
  ]
}
```

### Download and verify (owner decision)

1. The user selects "Download and verify" in the update popover.
2. The user chooses a folder (or the configured download folder).
3. Foliolight downloads the zip for the current architecture, computes SHA-256 while streaming, and compares it with the signed manifest and `SHA256SUMS`.
4. On success: the status shows "Verified," and the popover offers "Open folder" and short replacement instructions (close Foliolight, extract the new zip, copy the `data` folder over if using portable mode).
5. On failure: the partial file is deleted, and the status shows "Verification failed. Do not use this file."
6. Foliolight never extracts, runs, or replaces anything itself.

### Failure behavior

Any failure (offline, DNS, proxy, TLS, timeout, HTTP error, bad signature) sets the status icon to "Could not check" with a reason in the tooltip and the time of the last successful check. No dialog, no retry storm, no impact on any other function.

## Manual update path for locked-down agencies

1. On a connected machine, download the zip, `SHA256SUMS`, and `SHA256SUMS.minisig` from GitHub Releases.
2. Verify with any of:
   - `foliolight-cli update verify <zip>` from an existing trusted copy (works offline),
   - `Get-FileHash -Algorithm SHA256 <zip>` compared against `SHA256SUMS`, plus `minisign -Vm SHA256SUMS -P <published key>`,
   - `gh attestation verify <zip> --repo <org>/foliolight`.
3. Transfer through approved media.
4. Update AppLocker or WDAC rules if they are hash-based (publisher-based rules need no change).
5. Replace the folder contents, keeping `data\` for portable mode.

## FIPS and cryptography

| Use | Algorithm | Notes |
| --- | --- | --- |
| TLS for update check | Operating system Schannel | Follows the OS FIPS policy setting. Recommended so agencies do not depend on a bundled TLS library. |
| Release integrity | SHA-256 | FIPS-approved hash algorithm. |
| Release and pack signatures | Ed25519 (minisign) | EdDSA is included in FIPS 186-5, but the implementation is not a validated cryptographic module. Agencies that require validated modules can rely on Authenticode verification by Windows and on SHA-256 hashes checked with OS tools. |
| Comment fingerprints and file hashes | SHA-256 | Change detection only, not a security function. |
| Encryption at rest | None in v1 | Rely on OS volume encryption and NTFS permissions. |

## IT administrator guide (to be written in Phase 1)

Contents: what the app does and does not do, files and folders it writes, network behavior and host names, policy file reference, AppLocker and WDAC rule examples, hashes and signature verification, WebView2 requirements, uninstall (delete the folder and the `%APPDATA%\Foliolight` and `%LOCALAPPDATA%\Foliolight` folders).

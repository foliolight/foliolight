# Specification: Schema Pack v1

Status: Draft v0.1. Spec ID: `foliolight/schema-pack/v1`.

## 1. Purpose

A schema pack lets an industry partner, agency, or community member publish ready-made view schemas for a package format. An agency drops a pack into a folder (or installs it with the CLI) and gets a ready-made view.

**Packs never contain executable code.** They contain only data files from a strict allowlist.

## 2. Layout

A pack is a folder. Zip distribution is planned with archive support in Phase 7; until then, partners publish a zip that users extract and Foliolight installs from the folder.

```
example-publisher.oscal-ssp-compact/
  pack.yaml                 Manifest (required)
  schemas/
    ssp-compact.view.yaml
    poam-compact.view.yaml
  docs/
    README.md               Plain Markdown shown in the pack details screen
  SHA256SUMS                Hash of every file except SHA256SUMS and its signature (required)
  SHA256SUMS.minisig        Detached minisign signature over SHA256SUMS (optional)
```

### Allowed file types

| Extension | Purpose |
| --- | --- |
| `.yaml`, `.yml`, `.json` | Manifest and schemas |
| `.md`, `.txt` | Documentation (rendered without raw HTML) |
| `.png` | Documentation images (size-limited) |
| `SHA256SUMS`, `SHA256SUMS.minisig` | Integrity and signature |

Any other file, any symlink, any hidden file, or any path outside the pack folder causes the pack to be **rejected**.

### Limits

| Limit | Default |
| --- | --- |
| Total pack size | 5 MB |
| Number of files | 200 |
| Single schema file | 1 MB |
| Folder depth | 4 |

## 3. Manifest

```yaml
apiVersion: foliolight/schema-pack/v1
kind: SchemaPack
metadata:
  id: example-publisher/oscal-ssp-compact   # publisher/name, lowercase, [a-z0-9-]
  name: Compact OSCAL views
  version: 2.1.0                            # semantic version
  description: Shorter SSP and POA&M views for quick reviews.
  license: Apache-2.0                       # SPDX identifier, must pass dependency policy
  publisher:
    name: Example Publisher
    url: https://example.com                # display only, never fetched
    contact: security@example.com
  provenance:
    sourceRepository: https://example.com/packs/oscal-ssp-compact
    commit: 3f2c9a1
    builtAt: 2026-10-01T12:00:00Z
requires:
  foliolight: ">=1.0.0, <2.0.0"
  viewSchemaApi: foliolight/view-schema/v1
schemas:
  - path: schemas/ssp-compact.view.yaml
  - path: schemas/poam-compact.view.yaml
signing:
  keyId: "0123456789ABCDEF"   # minisign key ID, illustrative placeholder
```

Rules:

- Every schema's `metadata.id` must start with the pack's `publisher/` namespace.
- The `publisher` segment `builtin` and `foliolight` are reserved for the project.

## 4. Namespacing, versions, and conflicts

- Schema IDs are globally unique as `publisher/name`. Two packs cannot define the same schema ID; installing a second pack that does is rejected unless it is a newer version of the same pack.
- Several versions of one pack can be installed side by side. The highest compatible version is active unless the user pins one.
- When schemas from different packs match the same file, [view schema precedence](view-schema.md#10-precedence-and-selection) applies, and the user can choose with **View as** or bind a schema permanently.

## 5. Installation and discovery

- Installed packs live in `<config dir>/packs/<publisher>/<name>/<version>/`. See [app-config](app-config.md#2-config-location) for the config directory.
- **Drop folders**: the config can list folders (for example, a shared team folder) that are scanned for packs at startup. Packs there are validated and verified like installed packs but are never modified.
- Installation copies the pack after validation and hash verification. It never runs anything.

## 6. Trust model

| Trust level | Meaning | UI treatment |
| --- | --- | --- |
| Built-in | Ships inside Foliolight. | No badge |
| Signed and trusted | `SHA256SUMS.minisig` verifies against a key in the user's or admin's trust store. | "Verified: Example Publisher" |
| Hash-pinned | No trusted signature, but the hash of `SHA256SUMS` matches an entry in the trust store (useful for agency-vetted packs). | "Approved by your organization" |
| Unsigned | Hashes match `SHA256SUMS` but nothing vouches for the publisher. | Warning badge; installation requires `--allow-unsigned` or explicit confirmation |

Any hash mismatch means the pack is **rejected**, regardless of trust level.

### Trust store

```yaml
apiVersion: foliolight/trust/v1
kind: TrustStore
keys:
  - publisher: example-publisher
    keyId: "0123456789ABCDEF"
    publicKey: "RW...illustrative-placeholder..."
    addedAt: 2026-10-02T09:00:00Z
    status: active            # active | revoked
pinnedPacks:
  - id: example-publisher/oscal-ssp-compact
    version: 2.1.0
    sha256sums: 9b1d...e2       # SHA-256 of the SHA256SUMS file
```

- The trust store lives in the config directory. An admin can preseed it in the policy location and lock it (see [app-config](app-config.md#5-admin-policy)).
- A key with `status: revoked` causes all packs signed with it to drop to "Unsigned" with a revocation warning.
- Signatures use minisign (Ed25519) by default (ASSUMPTION A-17). Sigstore is a community open question.

## 7. What "no code" means, enforced

- File type allowlist and structural validation (Section 2).
- Schemas use only the fixed formatter list and FPath, which has no functions or expressions.
- Markdown documentation renders with raw HTML disabled and no remote images.
- URLs in manifests are display-only and never fetched.

## 8. Validation

`foliolight-cli pack validate <folder>` checks the layout, allowlist, limits, manifest schema, every view schema, namespace rules, `SHA256SUMS`, and the signature if present. It reports the resulting trust level.

## 9. Publishing guidance for partners

1. Start from `packs/examples/` in the Foliolight repository.
2. Keep views vendor-neutral where possible, and name extension properties clearly.
3. Publish the pack source in a public repository with tagged releases.
4. Sign `SHA256SUMS` with a minisign key, publish the public key on the partner's own site and in the pack README, and keep the secret key offline.
5. Run `foliolight-cli pack validate` in CI.

Curated pack listings and governance are discussed in [open questions](../community/open-questions.md#5-schema-pack-governance).

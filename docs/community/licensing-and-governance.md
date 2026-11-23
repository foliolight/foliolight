# Licensing, Contribution Terms, Trademark, and Governance

Status: Draft v0.1. This document explains choices in plain language. It is not legal advice; partner legal teams should review it before public launch.

## License choice

### Goals

Anyone may use, fork, modify, redistribute, and use Foliolight internally, commercially or not, with minimal restrictions. Anyone with good intent may contribute. Government and corporate legal teams should be able to approve it quickly.

### Comparison in plain language

| Topic | Apache-2.0 | MIT | BSD-3-Clause |
| --- | --- | --- | --- |
| What users may do | Use, modify, distribute, sublicense, commercial use | Same | Same |
| What users must do | Keep the license and NOTICE, mark changed files, keep attribution | Keep the copyright and license text | Keep the copyright, license text, and disclaimer |
| **Patent grant** | **Yes, explicit.** Each contributor grants a patent license for their contributions. If someone sues claiming the project infringes their patent, their patent license ends. | No explicit grant (an implied license may exist, but it is uncertain) | No explicit grant |
| **Contribution terms** | **Built in.** Section 5 says contributions are under the same license unless stated otherwise ("inbound equals outbound"). | Not addressed | Not addressed |
| **Trademarks** | **Explicitly excluded** (Section 6): the license gives no right to use the project name. | Not addressed | Clause 3 forbids using contributors' names to endorse derived products, which is narrower |
| Length and readability | Long, but well understood | Very short | Short |
| Government and corporate acceptance | Very widely approved; common in federal open-source projects and major foundations | Very widely approved | Widely approved |
| GPL compatibility | Compatible with GPLv3, not GPLv2-only | Compatible with both | Compatible with both |

### Recommendation: Apache-2.0 (confirmed)

Apache-2.0 is the right fit for a multi-company collaboration:

1. **Patent protection** matters when CSPs, vendors, and agencies contribute. Each contributor's patent license reassures everyone that a contributor cannot later assert patents against users of their own contributions.
2. **Section 5** gives clear contribution terms without a separate signed agreement.
3. **Section 6** already says the license does not grant trademark rights, which supports a lightweight trademark policy.
4. The NOTICE mechanism gives a clean, standard way to handle attribution.

When would MIT be better? If the project needed GPLv2-only compatibility or the absolute shortest license. Neither applies: we do not take GPL dependencies, and the patent and contribution benefits outweigh brevity.

### Disclaimer and limitation of liability

Apache-2.0 Sections 7 (Disclaimer of Warranty) and 8 (Limitation of Liability) provide the standard legal protections. The README adds a short plain-language disclaimer:

> Foliolight is provided "as is," without warranty of any kind. It is not an official government product, is not endorsed by any government agency or program, and is not a substitute for official review of any authorization package. See the LICENSE file for the full warranty disclaimer and limitation of liability.

The full license text is the official text from apache.org, copied unmodified into `LICENSE` and `LICENSES/Apache-2.0.txt`.

## Contribution terms: DCO instead of a CLA

| Option | What contributors do | Pros | Cons |
| --- | --- | --- | --- |
| Apache-2.0 Section 5 only | Nothing extra | Zero friction | No explicit per-commit record that the contributor had the right to contribute |
| **Section 5 plus DCO sign-off (recommended)** | Add `Signed-off-by:` to each commit with `git commit -s` | Explicit, per-commit certification; no documents to sign; widely used (Linux kernel, many CNCF projects); easy CI enforcement | Contributors must remember `-s` (CI reminds them and explains how to fix) |
| CLA | Sign an agreement once | Strongest legal record; allows relicensing | Friction; legal review at every contributing company; disliked by many contributors; requires a legal entity to hold it |

**Recommendation:** Apache-2.0 Section 5 plus the Developer Certificate of Origin 1.1, enforced in CI. No CLA and no separate signed documents. The official DCO text is in the [DCO](../../DCO) file.

### AI-assisted contributions

Most contributors are expected to use AI coding assistants. That is welcome, with these expectations:

- The contributor is responsible for the contribution as if they wrote it by hand, and certifies the DCO as usual.
- The contributor must understand the change well enough to explain it in review.
- Do not submit generated content that reproduces third-party code or text whose license is unknown or incompatible.
- Keep pull requests small and focused. Include tests.
- Disclosure of AI assistance is encouraged in the pull request description but not required.

## Trademark policy (lightweight)

See [TRADEMARKS.md](../../TRADEMARKS.md). Summary:

- The license does not grant rights to the Foliolight name or logo.
- Unmodified redistribution may use the name.
- Modified versions and forks should use a different name, and may say "based on Foliolight."
- Do not use the name in a way that suggests endorsement by the project or by any government agency.
- The project itself does not use government marks (for example, program names or seals) in its name, logo, or UI, and refers to government programs only as neutral context.

## Copyright line

Recommended wording, used in NOTICE and SPDX headers:

```
Copyright The Foliolight Contributors
```

- No employer-specific copyright holder. Contributors (or their employers) keep copyright in their contributions; "The Foliolight Contributors" is a collective description, which is common practice in community projects.
- No year, to avoid churn. Git history provides the dates.
- Contributors who must name their employer may add a separate `SPDX-FileCopyrightText` line for files they substantially author.

## SPDX identifiers and REUSE

Every source file starts with:

```
// SPDX-FileCopyrightText: The Foliolight Contributors
// SPDX-License-Identifier: Apache-2.0
```

(Comment syntax varies by language.) Files that cannot carry comments (JSON fixtures, images, generated files) are covered by `REUSE.toml`. License texts live in `LICENSES/`. `reuse lint` runs in CI so the project stays [REUSE](https://reuse.software/)-compliant.

Third-party texts that the project includes but does not license itself:

| File | License |
| --- | --- |
| `CODE_OF_CONDUCT.md` | Contributor Covenant 2.1, licensed CC-BY-4.0 |
| `DCO` | Verbatim copying permitted, changes not allowed (recorded as `LicenseRef-DCO`) |

## Dependency license policy

Only permissive, Apache-2.0-compatible licenses are allowed in anything we ship.

### Allowed

`Apache-2.0`, `Apache-2.0 WITH LLVM-exception`, `MIT`, `MIT-0`, `BSD-2-Clause`, `BSD-3-Clause`, `ISC`, `Zlib`, `0BSD`, `BSL-1.0`, `Unicode-3.0`, `Unicode-DFS-2016`, `CC0-1.0`, and dual licenses where one option is allowed (for example, `MIT OR Apache-2.0`).

### Disallowed

- Strong copyleft: `GPL-*`, `AGPL-*`.
- Weak copyleft: `LGPL-*`, `EPL-*`, `CDDL-*`, `EUPL-*`, and `MPL-2.0` (ASSUMPTION A-04: disallowed unless the maintainers approve a documented exception).
- Source-available or non-commercial: `SSPL`, `BUSL`, `Elastic-2.0`, `CC-BY-NC-*`, Commons Clause, and "JSON license" style use restrictions.
- No license, unknown license, or custom license without maintainer review.

### Known special case: WebView2 fixed-version runtime

The fixed-version Windows Server build bundles Microsoft's WebView2 runtime, which is proprietary software redistributed under Microsoft's terms. It is not a library we link; it is a separately licensed component. The maintainers must explicitly approve this exception (or drop the fixed-version build and require agencies to install the WebView2 runtime themselves). This is listed as an open decision.

### Bundled third-party data (published schemas)

Foliolight bundles copies of schemas that programs publish, starting with the JSON schemas FedRAMP publishes (see [bundled-program-schemas](../specs/bundled-program-schemas.md)). These are data files, not dependencies, but they are redistributed, so the same care applies:

- Their redistribution terms must be confirmed with the publisher and recorded before they are committed (ASSUMPTION A-32). The publisher's schemas page does not state terms.
- Each file is recorded with its real license or public-domain status in `REUSE.toml` and `NOTICE`, and in the pack's `PROVENANCE.yaml` with source URL, retrieval date, and SHA-256.
- Files are copied unmodified. If redistribution cannot be confirmed, the pack ships views only.

### Enforcement

| Tool (candidates confirmed in Phase 0) | What it checks |
| --- | --- |
| `cargo-deny` | Rust crate licenses against the allowlist, advisories, banned crates, allowed registries |
| An npm license checker (for example, `license-checker-rseidelsohn`) | npm package licenses against the allowlist |
| `reuse lint` | SPDX headers and license files for project files |
| `cargo-about` plus an npm equivalent | Generates `THIRD_PARTY_LICENSES.txt` with full license texts and NOTICE contents of every shipped dependency |

CI fails on any disallowed or unknown license. Dev-only tools (fixture generator, test tools) should also be permissive but are not shipped.

## Governance summary

See [GOVERNANCE.md](../../GOVERNANCE.md) for the full text.

| Topic | Rule |
| --- | --- |
| Roles | Contributor, reviewer, maintainer, org owner |
| Becoming a maintainer | Nominated by a maintainer after sustained, good-faith contribution; approved if no maintainer objects within 7 days and at least two other maintainers approve |
| Decisions | Lazy consensus on pull requests and discussions; a simple majority vote of maintainers if consensus fails |
| Releases | Two maintainer approvals; signing only in the protected release environment |
| Org owners | At least three people from at least three different organizations |
| Vendor neutrality | No vendor-specific features in core; vendor- or program-specific views go in schema packs; no maintainer may block a change solely for commercial reasons |
| Security-sensitive code | CODEOWNERS requires a maintainer review for parsers, launcher, updater, config and policy, CI and release workflows |
| Removing maintainers | Inactive 12 months: moved to emeritus. Code of Conduct violations: per the enforcement guidelines |
| Name and org ownership | Held by the maintainer council; goal of transfer to the trade organization or a neutral foundation |

## Sunset plan

Foliolight is a stopgap. A graceful end is part of the plan.

1. **Trigger:** maintainers agree the mission is served (for example, enterprise tools offer equivalent vendor-neutral viewing), or fewer than two active maintainers for six months.
2. **Notice:** announce at least 90 days ahead in the README, Discussions, and release notes.
3. **Final release:** includes all known security fixes; the update check reports "This project is archived" instead of offering updates.
4. **Archive:** the repository is set to read-only. Releases, hashes, signatures, SBOMs, public keys, and docs remain available.
5. **Handoff:** a willing steward may continue the project under Apache-2.0. Use of the name requires agreement from the maintainer council per TRADEMARKS.md; otherwise forks rename.
6. **Signing identity:** retired after the final release; no new artifacts are signed with it.

# Open Questions for the Community

Status: Open for discussion. Each question lists options, tradeoffs, and a recommended default. The project proceeds with the recommended default unless the community decides otherwise. Please discuss in GitHub Discussions (one thread per question) once the repository is public.

---

## 1. Distribution format and code signing

**Question:** What should we ship, and which signing approach is most likely to be accepted by agency IT? Who holds the signing identity for a community-maintained project?

| Option | Pros | Cons |
| --- | --- | --- |
| Signed portable zip | No installer, no admin, works from any approved folder, easy to hash-allowlist | Some catalogs require an installer; AppLocker blocks user-writable paths by default |
| Signed MSI (per-user) | Fits software catalogs and deployment tools | More build complexity; still needs signing |
| MSIX | Clean install and uninstall | Container model complicates arbitrary folder and share access; requires package identity; some agencies block sideloading |
| Signed EXE installer | Familiar | Installers are a larger attack surface and are often flagged |

| Signing approach | Pros | Cons |
| --- | --- | --- |
| Azure Trusted Signing | Low cost, no key custody (managed HSM), GitHub OIDC integration, SmartScreen-recognized | Requires an eligible, validated organization; short-lived certificates (rules must target publisher, not thumbprint) |
| OV certificate in cloud HSM | Stable thumbprint | Cost and key custody by some organization |
| EV certificate | Strongest identity vetting | Highest cost; no longer an instant SmartScreen advantage |
| Sigstore | Free, transparent, great provenance | Not understood by Windows, AppLocker, or WDAC; complement only |

**Signing identity holder options:** the CSP trade organization; a neutral foundation (for example, a Linux Foundation project or similar host); a founding partner company; an individual maintainer.

**Recommended default:** Signed portable zip as the primary artifact, MSI later if agencies ask. Azure Trusted Signing held by a neutral legal entity (the trade organization or a foundation), used only from the protected release environment, plus Sigstore-based provenance attestations. Never an individual's personal identity.

---

## 2. Portable versus installed

**Question:** Does portable software pass typical agency approval, or do agencies require an installer, an allowlist hash, or a vetted catalog entry?

| Option | Pros | Cons |
| --- | --- | --- |
| Portable only | Simplest, no admin, nothing to uninstall | Blocked by AppLocker default rules from user paths; some policies prohibit unmanaged executables |
| Portable plus published hashes and publisher identity | Lets IT create hash or publisher allow rules and place the folder in an approved path | Hash rules must be updated every release |
| Installer for catalogs | Fits enterprise deployment tooling | More to build and sign; admin may be required for machine-wide installs |

**Recommended default:** Portable plus published hashes, a stable publisher identity, and an IT administrator guide. Ask agency participants which of their approval processes accept this, and add a per-user MSI if several require it.

---

## 3. Update model

**Question:** Notify-only or self-update? Should hashes and an SBOM be published with each release?

| Option | Pros | Cons |
| --- | --- | --- |
| Notify only | No download code; nothing for agencies to worry about | More manual steps for users |
| Download and verify, user replaces (current plan, owner decision) | Convenient; verification built in; never executes anything | Download code to maintain; still manual replacement |
| Full self-update | Most convenient | Replacing executables in place conflicts with allowlisting and change control; largest risk |

**Recommended default:** Download and verify (already decided by the project owner), with the check off until opted in, disableable by policy. Yes: publish SHA-256 hashes, signatures, CycloneDX SBOM, third-party license inventory, and provenance attestations with every release.

---

## 4. Governance

**Question:** How are maintainers added and releases approved, given anyone with good intent may become a maintainer? Who owns the project name and repository organization over time?

| Option | Pros | Cons |
| --- | --- | --- |
| Single benevolent maintainer | Fast | Single point of failure; vendor-neutrality concerns |
| Maintainer council by lazy consensus (recommended) | Lightweight, inclusive | Needs clear tie-breaking |
| Foundation-hosted project | Neutral ownership of name and assets | Overhead; may be more than a stopgap needs |

**Recommended default:** See [GOVERNANCE.md](../../GOVERNANCE.md). Maintainers nominated by an existing maintainer after sustained contribution, approved by lazy consensus with no objection from at least two other maintainers in 7 days. Releases need two maintainer approvals. GitHub org owners come from at least three different organizations. The name and org are held by the maintainer council, with a goal of transferring them to the trade organization or a neutral foundation if one agrees.

---

## 5. Schema pack governance

**Question:** Who curates packs, how are they validated, and how do agencies trust them?

| Option | Pros | Cons |
| --- | --- | --- |
| No curation: publishers distribute their own | Zero project overhead | Agencies must evaluate each publisher alone |
| Community index (a list of packs with publisher keys and hashes, reviewed by maintainers) | Discoverability; a second set of eyes | Implies endorsement unless worded carefully; review effort |
| Agency-run allowlists | Agencies keep control | Each agency repeats the work |

**Recommended default:** Publishers distribute their own signed packs. The project maintains a **community index** listing pack ID, publisher, public key, source repository, and hashes, with an explicit statement that listing means "validates and follows the format," not "endorsed or correct." Agencies pin packs by hash or trust publisher keys through the admin policy. Pack signing uses minisign by default; Sigstore keyless signing is an alternative to evaluate (ASSUMPTION A-17).

---

## 6. Comment portability

**Question:** Should there be an interoperable comment format so findings can move between tools?

| Option | Pros | Cons |
| --- | --- | --- |
| Foliolight format only (current plan) with Markdown, CSV, JSON export | Simple | No round trip with other tools |
| Export to OSCAL assessment-results observations or findings | Uses an existing standard; GRC tools could import | Mapping reviewer comments to observations is not exact; extra work |
| Adopt the W3C Web Annotation model | Mature annotation standard | Unfamiliar in this domain |

**Recommended default:** Keep the Foliolight format, publish its spec, and add an **OSCAL assessment-results export** after 1.0 if partners confirm they would import it.

---

## 7. Platform scope

**Question:** Windows first, but what is the plan for macOS and Linux?

| Option | Pros | Cons |
| --- | --- | --- |
| Windows only | Focus | Excludes some reviewers and CSP staff |
| Community-supported macOS and Linux builds after 1.0 | Tauri supports them; low cost if volunteers test | Different web engines (WebKit) mean rendering and accessibility differences; signing and notarization on macOS |
| Fully supported cross-platform | Broadest reach | Triples the test matrix for a stopgap |

**Recommended default:** Windows-only through 1.0. Keep the core platform-neutral. Offer community-supported macOS and Linux builds after 1.0 if volunteers commit to testing them.

---

## 8. Offline, air-gapped, and crypto requirements

**Question:** What offline behavior and FIPS or other crypto requirements might agencies impose?

**Current plan:** Air-gapped mode is the update check turned off (by user or policy), which guarantees zero network activity. TLS uses the OS (Schannel) and follows OS FIPS policy. Signatures use Ed25519, which is in FIPS 186-5 but not from a validated module; agencies can rely on Authenticode and SHA-256 instead.

**Questions for agencies:** Is a validated cryptographic module required for signature verification in a read-only viewer? Is the update check acceptable at all, or should builds for some environments compile it out entirely?

**Recommended default:** Keep the current plan. Offer a build flag that compiles out the update checker entirely if an agency needs that for approval.

---

## 9. Sunset plan

**Question:** Since this is a stopgap, how should the project be archived or handed off gracefully?

**Recommended default** (also in [licensing-and-governance](licensing-and-governance.md#sunset-plan)):

- Define the sunset trigger: when maintainers agree the project's purpose is served (for example, enterprise tools provide equivalent vendor-neutral viewing), or when there are fewer than two active maintainers for six months.
- Announce at least 90 days ahead. Publish a final release with all known security fixes.
- Turn off the update check in the final release (or point the manifest at a final "archived" notice).
- Archive the repository read-only, keep releases, hashes, signatures, and public keys available.
- Offer handoff to a willing steward (trade organization, foundation, or agency) under the same license, with the name only if the trademark policy conditions are met.

---

## 10. Additional questions raised during planning

| Question | Options | Recommended default |
| --- | --- | --- |
| Allow MPL-2.0 dependencies? | Disallow; allow file-level weak copyleft by exception | Disallow by default; documented exception process (ASSUMPTION A-04) |
| Update check default? | Off until opt-in; on by default | Off until opt-in on first run (ASSUMPTION A-15) |
| Comment store encryption? | None; DPAPI per user; password-based | None in v1; revisit if agencies require it |
| Compile-out build for strict environments? | Single build; extra build without network code | Offer if requested by an agency |
| Bundle OSCAL JSON Schemas for informational validation? | Yes; no | Yes in Phase 7, after license review |
| UI prototyping tool? | Penpot (open source); Figma (proprietary, account required); HTML prototype only | Penpot or Figma for round 1, Storybook for round 2 (see [ui-prototyping](../design/ui-prototyping.md)) |
| Who publishes program-specific dashboard packs (for example, vulnerability report exposure classes and impact ratings)? | The program owner; a partner; the community example only | Community example pack in the repo; partners publish maintained packs with field names confirmed against the real format (ASSUMPTION A-25) |
| Cross-document charts (trends across versions, comparisons across CSPs)? | Not planned; post-1.0 | Post-1.0. Needs library-wide parsing and possibly an in-memory SQLite engine |

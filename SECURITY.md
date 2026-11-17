# Security Policy

## Reporting a vulnerability

Please report vulnerabilities **privately** using GitHub private vulnerability reporting: open the repository's **Security** tab and choose **Report a vulnerability**.

Do not open a public issue for security problems.

If you cannot use GitHub, use the security contact listed in [GOVERNANCE.md](GOVERNANCE.md) (to be established before public launch).

### Please include

- Affected version and Windows version.
- A description of the issue and its impact.
- Steps to reproduce, using **synthetic** files only. Never send real authorization packages or real system data. If a real file triggered the problem, describe its structure or create a minimal synthetic file that reproduces it.

## What to expect

Foliolight is maintained by volunteers. These are goals, not guarantees:

| Step | Target |
| --- | --- |
| Acknowledge report | Within 5 business days |
| Initial assessment | Within 10 business days |
| Fix for critical severity | Within 14 days of confirmation |
| Fix for high severity | Within 30 days of confirmation |
| Fix for medium and low severity | Next planned release |

We follow coordinated disclosure. We will agree on a disclosure date with you, credit you in the advisory if you wish, and publish a GitHub Security Advisory with the fixed release.

## Scope

In scope:

- The Foliolight application and CLI, including parsers, rendering, external launching, comments, configuration, and the update checker.
- Release integrity: signatures, hashes, update manifests, and build pipeline.
- Schema pack validation and trust checks.

Out of scope:

- Vulnerabilities in Windows, WebView2, or default applications that Foliolight launches, unless Foliolight's behavior makes them exploitable. Please report those to the vendor.
- The content or correctness of any authorization package.
- Third-party schema packs (report to their publisher), except where a pack bypasses Foliolight's validation.

## Safe harbor

We will not pursue or support legal action against researchers who act in good faith, avoid privacy violations and data destruction, use only synthetic data, and give us reasonable time to fix issues before disclosure.

## Verifying releases

Each release publishes SHA-256 hashes, a signature over the hashes, an SBOM, and build provenance attestations. See the [release plan](docs/release/release-and-distribution.md) for verification steps.

# Governance

Status: Draft. To be ratified by the founding maintainers before public launch.

## Mission

Provide a free, local, vendor-neutral viewer for machine-readable authorization packages, as a stopgap and demonstration, and retire it gracefully when its purpose is served.

## Roles

| Role | Who | Responsibilities |
| --- | --- | --- |
| Contributor | Anyone who submits issues, pull requests, schemas, docs, or reviews | Follow CONTRIBUTING.md and the Code of Conduct |
| Reviewer | Contributors trusted to review specific areas | Review pull requests; cannot merge alone |
| Maintainer | Listed in `MAINTAINERS.md` (to be created) | Merge pull requests, triage, approve releases, enforce the Code of Conduct |
| Org owner | At least three maintainers from at least three different organizations | Administer the GitHub organization, release environment, and signing integration |

## Becoming a maintainer

1. An existing maintainer nominates a contributor who has shown sustained, good-faith contributions and sound judgment.
2. The nomination is posted to the maintainers for 7 days.
3. The nominee is approved if at least two other maintainers approve and no maintainer objects. Objections are discussed; if unresolved, a majority vote of maintainers decides.

Anyone with good intent may become a maintainer. Employer does not matter, but no single organization should hold a majority of maintainers or org owners; if it does, new nominations from other organizations are prioritized.

## Decisions

- **Lazy consensus** on pull requests and discussions: a proposal stands if no maintainer objects within a reasonable time (at least 72 hours for significant changes).
- If consensus fails, a **simple majority** of maintainers who vote within 7 days decides.
- Changes to this governance document, the license, or the trademark policy need a two-thirds majority of maintainers.

## Releases

- A release requires approval from **two maintainers**.
- Signing happens only in the protected release environment; no individual holds signing keys.
- The release process is documented in [docs/release/release-and-distribution.md](docs/release/release-and-distribution.md).

## Vendor neutrality

- No vendor-specific or program-specific features in the core. Such views belong in schema packs.
- No employer branding in the product, docs, or copyright lines (use "The Foliolight Contributors").
- A maintainer may not block a change solely for the commercial benefit of their employer.
- Maintainers disclose conflicts of interest when reviewing changes that affect their employer.

## Security-sensitive code

`.github/CODEOWNERS` requires maintainer review for parsers, the external launcher, the updater, config and policy, and CI and release workflows.

## Inactivity and removal

- Maintainers inactive for 12 months move to emeritus status and can return by asking.
- Maintainers may be removed for Code of Conduct violations under its enforcement guidelines, or by a two-thirds vote of other maintainers.

## Contacts

To be established before public launch:

- Conduct reports: (contact to be defined)
- Security reports: GitHub private vulnerability reporting (primary); backup contact to be defined

## Ownership of name and assets

The maintainer council holds the project name, GitHub organization, and signing integration on behalf of the community. The goal is to transfer them to a neutral steward (the CSP trade organization or a foundation) if one agrees.

## Sunset

The sunset plan is in [docs/community/licensing-and-governance.md](docs/community/licensing-and-governance.md#sunset-plan).

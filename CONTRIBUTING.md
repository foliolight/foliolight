# Contributing to Foliolight

Thank you for helping. Anyone acting in good faith is welcome to contribute: code, view schemas, documentation, accessibility testing, synthetic samples, and reviews.

## Before you start

- Read the [project plan](docs/PLAN.md), especially the principles and non-goals. Changes that add network access, background activity, writes to package folders, or runtime code loading will not be accepted.
- For anything larger than a small fix, open an issue or discussion first so we can agree on the approach.
- Follow the [Code of Conduct](CODE_OF_CONDUCT.md).

## Never share real package data

Do not attach real authorization packages, real system details, or real findings to issues, pull requests, or discussions, even if they seem harmless. Reproduce problems with the synthetic samples in `fixtures/`, or describe the structure without the content. If you accidentally post sensitive data, contact the maintainers immediately so it can be removed.

## Contribution terms

- Foliolight is licensed under [Apache-2.0](LICENSE). Under Section 5 of the license, contributions you submit are under the same license.
- Every commit must be **signed off** under the [Developer Certificate of Origin 1.1](DCO). A sign-off is a line at the end of the commit message certifying that you have the right to submit the work:

  ```
  Signed-off-by: Your Name <you@example.com>
  ```

  Add it automatically with `git commit -s`. To fix a branch that is missing sign-offs: `git rebase --signoff main`, then force-push your branch.
- No CLA or other signed document is required.
- If your employer has rights to your work, make sure you are allowed to contribute it.

## AI-assisted contributions

AI coding assistants are welcome. You remain responsible for what you submit:

- Understand your change well enough to explain and defend it in review.
- Do not submit generated content that copies third-party code or text with an unknown or incompatible license.
- Include tests, and keep pull requests small and focused.
- Mentioning AI assistance in the pull request description is encouraged but not required.

## Pull request checklist

- [ ] Commits are signed off (`git commit -s`). Signed commits (GPG, SSH, or Sigstore gitsign) are required on `main`.
- [ ] New source files start with SPDX headers:

  ```
  // SPDX-FileCopyrightText: The Foliolight Contributors
  // SPDX-License-Identifier: Apache-2.0
  ```

- [ ] Tests added or updated. Code that parses input includes hostile-input fixtures and a fuzz target.
- [ ] Accessibility: keyboard operable, labeled for screen readers, no color-only meaning.
- [ ] Any new dependency is justified in the description and uses an allowed license (see the [dependency license policy](docs/community/licensing-and-governance.md#dependency-license-policy)).
- [ ] Documentation and specs updated.
- [ ] No new network access and no new file writes outside documented locations.

## Security-sensitive areas

Changes to parsers, the external launcher, the updater, config and policy handling, and CI or release workflows require review by a maintainer listed in `.github/CODEOWNERS`.

## Contributing view schemas and packs

View schemas and schema packs are data, not code, and are a great way to contribute without programming. See the [view schema spec](docs/specs/view-schema.md) and the [schema pack spec](docs/specs/schema-pack.md).

## Writing style for docs

- Plain language, short sentences.
- Do not use em dashes in public docs.
- Refer to government programs only as neutral context; never imply endorsement.

## Getting help

Open a discussion, or ask in your pull request. Maintainers are volunteers, so please be patient.

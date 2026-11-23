# Specification: Bundled Program Schemas (FedRAMP JSON Schemas)

Status: Draft v0.1. Added at owner direction on 2026-11-20, following [issue #2](https://github.com/foliolight/foliolight/issues/2).

## 1. Decision

Foliolight ships with the JSON schemas that FedRAMP publishes for machine-readable submission artifacts, plus ready-made views for them, in a **bundled pack**. A reviewer who opens one of these files gets a useful view and an informational structure check with no setup and no network access.

These schemas are **not required** reading or checking for anything in Foliolight: every file opens without them, and the whole bundled pack can be switched off (Section 8).

## 2. Design rules

| Rule | Why |
| --- | --- |
| The bundled schemas and views are **data in a pack**, not code in the core. The core engine knows nothing about FedRAMP. | Keeps the core program-neutral and lets other programs' schemas be added the same way (ASSUMPTION A-18 is narrowed, not dropped). |
| The pack is a normal schema pack with the built-in trust level, and it uses the project's reserved `foliolight/` namespace. | Same validation, hashing, and precedence rules as every other pack. |
| Schema files are **unmodified copies** of the published files, each recorded with its source and SHA-256. | Reviewers can trust the check reflects the published schema. |
| Nothing is downloaded at run time. New schema versions arrive in a Foliolight release, or from a pack the user adds. | Principles 1 and 2 (no network). |
| Results are informational and worded narrowly. | The check says how a file compares with a published schema. It does not say whether a submission is acceptable. |

## 3. What is bundled

The set published at <https://www.fedramp.gov/schemas/> on 2026-11-20 is eleven schema files, all date-stamped `2026-06-24`. The list is rechecked whenever the bundle is refreshed (Section 6).

| Published schema | Help for the reviewer | View treatment |
| --- | --- | --- |
| Vulnerability Detail Report | Non-accepted vulnerability detection and response activity | **Dashboard** (counts by disposition, exposure class, impact rating, due dates) plus tables |
| Accepted Vulnerability Info | Accepted vulnerability information | **Dashboard** and tables |
| Historical VER Activity | Historical vulnerability evaluation and reporting activity | **Dashboard** (activity over time) and tables |
| Certification Package Overview | Cloud service offering identification, properties, contacts, supporting documents | Document view |
| Ongoing Certification Report | Quarterly report covering the period since the previous one | Document view |
| Security Decision Record | Provider system submission | Document view |
| Significant Change Notification | Required information for a significant change | Document view |
| Incident Report | Initial, ongoing, and final incident reports in one schema | Document view with the report type shown |
| Advisor Information | Public information an advisory service publishes | Document view |
| Assessor Information | Public information an independent assessment service publishes | Document view |
| Common Definitions | Shared type definitions referenced by the other schemas | **No view.** Used only to resolve `$ref` for the structure check. |

Field names, categories, and labels in the views come from the real schemas. This replaces the illustrative names used in the synthetic example pack for these formats (see ASSUMPTION A-25, which still applies to category definitions that a schema does not spell out). The dashboards for the three vulnerability-related schemas cover the counts reviewers ask for: totals, accepted, false positive, and the exposure class and impact rating breakdowns where the schema defines them.

The first release of the pack generates Document views by inference from each schema and then hand-adjusts labels and ordering. Dashboards are built by hand and checked against synthetic data generated from the schema.

## 4. Detection and version choice

- A file's `$schema` value is matched with `jsonSchemaId` globs, for example `https://www.fedramp.gov/schemas/fedramp-vulnerability-detail-report-schema-*.json`, so any dated version matches one view (see [view-schema Section 4](view-schema.md#4-match-rules)). The value is compared as text and never fetched.
- ASSUMPTION A-31: files carry a `$schema` value in that form. This must be confirmed against the published schemas and real files before the globs are written. If files do not carry `$schema`, the views fall back to `rootKey` and `exists` rules on the schema's required properties, and the suggestion says "This looks like a vulnerability detail report" instead of "This is."
- **Dated versions:** schema file names carry a date. The pack keeps every dated version it has shipped (they are small), so an older file is still checked against the version it names. If the file names a version, that one is used. If it does not, the newest bundled version is used and the result says which.
- **Newer than the app:** if a file names a version newer than any in this copy of Foliolight, the check says so ("This file follows a newer schema version than this copy of Foliolight includes. The check used version 2026-06-24 and may report differences that are not real.") and offers to open the folder where the user can add a newer schema to a personal pack.

## 5. What the reviewer sees

- Opening a matching file suggests the view as in [opening-files](opening-files.md#3-choosing-a-view-the-first-time), labelled with where it comes from: "Views for FedRAMP JSON schemas (provided by Foliolight)."
- The structure check panel names the schema, its date, and says it is informational:

  ```
  Structure check (informational): 2 differences
  Compared with: published JSON schema "Vulnerability Detail Report", version 2026-06-24 (included with Foliolight)
    - Line 412: "severity" must be one of the values the schema lists. Found: "Sev-High".
    - Line 1,009: required field "detected-date" is missing.
  A file that differs from the published schema may still be acceptable.
  Foliolight cannot tell what a program requires. This is not a compliance determination.
  ```

- Differences never stop a file from opening, and a "fits, with differences" outcome is still shown in the usual way.
- The About screen for the pack shows the disclaimer in Section 7.

## 6. Provenance and refresh

Each bundled schema file is recorded in `PROVENANCE.yaml` inside the pack (an allowed `.yaml` file):

```yaml
- file: schemas/fedramp-vulnerability-detail-report-schema-2026-06-24.json
  title: Vulnerability Detail Report
  sourceUrl: https://www.fedramp.gov/schemas/fedramp-vulnerability-detail-report-schema-2026-06-24.json
  retrievedOn: 2026-11-20            # illustrative; the real date is recorded when files are copied
  sha256: "<hash of the file as retrieved>"
```

- A development-only script, `tools/refresh-bundled-schemas` (not shipped, not part of the app), fetches the published files, compares them with the bundled copies, and prepares a pull request. A maintainer reviews the diff. The app never fetches schemas.
- Refreshing the bundle is a step in the release runbook, and a scheduled CI job opens an issue when the published list or any file changes (the job runs in CI, never in the app).
- CI fails if a bundled schema file's hash does not match `PROVENANCE.yaml`, so copies cannot drift silently.
- Views must keep working against all bundled dated versions. A new version that renames a field adds a view variant that `extends` the previous one.

## 7. Names, marks, and licensing

- **Naming:** "FedRAMP" appears only to describe what the schemas are and where they come from, in the pack title, descriptions, and docs. No logos, seals, colors, or styling from the program are used, and the application's own name, logo, and chrome carry no program marks. The publisher's site states that the FedRAMP name and logo are the property of GSA, and the plan respects that (see [TRADEMARKS](../../TRADEMARKS.md)).
- **Disclaimer** shown in the pack details and the structure check help: "These views and the included copies of published schemas are provided by Foliolight for convenience. Foliolight is not affiliated with or endorsed by FedRAMP, GSA, or any government agency. The published schemas, not this tool, define what is required."
- **Licensing of the copies:** ASSUMPTION A-32: the published schema files may be redistributed unmodified inside an Apache-2.0 project. The schemas page does not state terms. Before the files are committed, the maintainers confirm the terms with the publisher (and record the answer), then record the files in `REUSE.toml` and `NOTICE` with their actual license or public-domain status. If redistribution is not confirmed, the pack ships without the copies and the user is guided to add them from a folder (see Section 9), while the views still ship.
- **No endorsement:** the structure check wording never says a file is "valid for submission," "compliant," or "accepted."

## 8. Optional by design

- A setting turns the bundled pack off, and policy can lock it: `bundledPacks: { fedramp-json: { enabled: true } }` (see [app-config](app-config.md#3-settings-file)). Off means no suggestions, no structure check, and no pack views for these files, which then open as generic JSON in Outline and Source.
- A reviewer can always choose another view or build their own.
- Locked-down environments that want nothing bundled beyond OSCAL can turn it off once in policy.

## 9. Adding newer schema versions yourself

Because the published schemas are plain files, a reviewer or administrator can add a newer one without waiting for a release: put the schema file in a personal pack folder (a `structure/` file listed by a small `pack.yaml`). It is verified like any pack and used for the check when it matches the file's `$schema`. Instructions go in the pack's documentation and the administrator guide.

## 10. Phasing

| Phase | Delivered |
| --- | --- |
| 2 | The bundled schema files and Common Definitions for the **structure check** and for `$schema` detection of these files; generic OSCAL-style suggestion text. `PROVENANCE.yaml` and the CI hash check. |
| 4 | The bundled pack proper: Document views for every schema, dashboards for the three vulnerability-related schemas, trust level and About screen, the on/off setting, and the refresh script. |
| 7 | Review of coverage with whoever publishes the schemas, and any views the community has refined. |

## 11. Testing

- Synthetic documents are **generated from each bundled schema**: valid ones, and variants with a missing required field, a wrong enum value, an extra field, and a `$schema` version newer than bundled. No real data is used.
- Detection tests cover `$schema` present, absent, and with an unknown future version.
- Dashboard counts for the three vulnerability-related schemas are checked against an independent script's counts on generated data.
- A test confirms that disabling the pack removes suggestions and the structure check.
- A hostile test confirms that `$ref` values pointing at remote URLs or local files are never followed (only the bundled Common Definitions resolve).
- A CI check verifies bundled file hashes against `PROVENANCE.yaml`.

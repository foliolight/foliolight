# Specification: View Schema v1

Status: Draft v0.1. Spec ID: `foliolight/view-schema/v1`.

## 1. Purpose

A view schema describes how a structured file should be displayed: which sections to show, which fields become table columns, how values are labeled and formatted, what is hidden, and how links and attachments behave. View schemas are plain files that people can read, diff, review, and share. They are **data, not code**.

Built-in views (including all OSCAL views) are written in this same format, so partners can override or extend them.

## 2. File format

- **YAML (canonical) or JSON.** Both are accepted. The schema builder writes YAML because it supports comments and produces clean diffs. JSON is accepted for tools that generate schemas.
- YAML must use the **safe subset**: no anchors or aliases, no custom tags, no merge keys, UTF-8 only, a single document per file.
- File extension: `.view.yaml`, `.view.yml`, or `.view.json`.
- Every schema is validated against a published JSON Schema: `schemas/spec/view-schema-v1.schema.json`.
- Unknown fields produce a warning and are ignored (forward compatibility). `--strict` validation turns warnings into errors.

## 3. Top-level structure

```yaml
apiVersion: foliolight/view-schema/v1
kind: ViewSchema
metadata:
  id: example-publisher/oscal-ssp-compact     # publisher/name, lowercase, [a-z0-9-]
  name: Compact SSP view
  version: 1.2.0                              # semantic version of this schema
  description: A shorter SSP view focused on control status.
  license: Apache-2.0                         # SPDX identifier
extends: builtin:oscal/ssp@^1                 # optional, see Section 9
match: { ... }                                # Section 4
priority: 100                                 # Section 10
display: { ... }                              # title and summary
datasets: [ ... ]                             # Section 6A: reusable record sets and derived categories
pages: [ ... ]                                # Section 6B: optional tabs, for example a dashboard page
sections: [ ... ]                             # Section 6 (used directly when there are no pages)
hidden: [ ... ]                               # FPath patterns hidden everywhere
links: { ... }                                # Section 8 (including cross-document links, 8A)
attachments: { ... }                          # Section 8
versionField: /report/generated-date          # optional: in-document date used for version ordering
fit: { ... }                                  # Section 14: what a file must contain for this view to apply
structureSchema: { ... }                      # Section 15: published schema for the informational structure check
```

## 4. Match rules

A schema applies to a file when its `match` block is satisfied. All listed conditions must hold (`all`), or at least one (`any`).

```yaml
match:
  formats: [json, yaml, xml]    # also csv
  all:
    - oscal:
        model: system-security-plan
        versions: ">=1.1.0, <2.0.0"
  any:
    - exists: "/system-security-plan/metadata/props[name=marking]"
    - fileGlob: ["*ssp*"]
  documentTypes: [SSP]          # optional: library document type
```

Available conditions:

| Condition | Meaning |
| --- | --- |
| `formats` | Detected format must be one of these. |
| `oscal` | OSCAL model and optional version range. |
| `rootKey` | JSON or YAML root key equals this value. |
| `rootElement` | XML root element, with optional `namespace`. |
| `jsonSchemaId` | The file's `$schema` value equals this string, or matches it as a glob (for example `https://example.org/schemas/*/vuln-report*.json`), so one view can cover several published versions. The value is only compared as text; it is never fetched. |
| `csvHeaders` | CSV only: `{ includes: [...] }` or `{ equals: [...] }`, compared case-insensitively with the header row. This is usually the only match rule a CSV view needs. |
| `exists` | An FPath resolves to at least one node. Evaluated on the first 64 KB only, to keep matching cheap. |
| `equals` | `{ path, value }`: an FPath resolves to this scalar. Same 64 KB rule. |
| `fileGlob` | File name glob. |
| `documentTypes` | Library document type. |

## 5. FPath: the path language

FPath is a deliberately small, bounded path language that works the same on JSON, YAML, and XML through the unified node model. It is **not** JSONPath or XPath, and it has no functions, arithmetic, or expressions, so evaluating a schema can never run code or take unbounded time.

| Syntax | Meaning | Example |
| --- | --- | --- |
| `/a/b` | Absolute path from document root | `/system-security-plan/metadata/title` |
| `a/b` | Relative to the current context node | `control-id` |
| `*` | Any single child | `/catalog/groups/*` |
| `**` | Any descendants (depth-limited to 32) | `**/remarks` |
| `[n]` | Array index or nth element (0-based) | `parties[0]` |
| `[key=value]` | Child objects whose `key` equals `value` | `props[name=implementation-status]` |
| `@name` | XML attribute; for JSON and YAML, same as a key | `@uuid` |
| `#text` | XML element text content | `title/#text` |

FPath evaluates over the node model, so JSON `props[name=x]/value` and XML `prop[@name=x]/@value` are both supported. Built-in OSCAL schemas use format-specific paths where the two formats differ, selected with a `formats` list on a section.

## 6. Sections and layouts

```yaml
sections:
  - id: overview
    label: System overview
    source: /system-security-plan/system-characteristics
    layout: fields
    fields:
      - { path: system-name, label: System name }
      - { path: description, label: Description, format: markdown }
      - { path: security-sensitivity-level, label: Sensitivity, format: enum-label,
          enumLabels: { low: Low, moderate: Moderate, high: High } }

  - id: controls
    label: Control implementations
    source: /system-security-plan/control-implementation/implemented-requirements/*
    layout: table
    rowKey: control-id
    anchor: { kind: oscal-control-id, path: control-id }
    columns:
      - { path: control-id, label: Control, format: control-id, sort: natural, width: narrow }
      - { path: "props[name=implementation-status]/value", label: Status, format: badge,
          badges: { implemented: success, partial: warning, planned: info, not-applicable: neutral } }
      - { path: "statements/*", label: Statements, format: count }
    defaultSort: { column: Control, direction: asc }
    groupBy: { path: control-id, format: control-family }
    filters:
      - { column: Status, type: enum }
      - { column: Control, type: text }
    detail:
      layout: fields
      fields:
        - { path: "statements/*/by-components/*/description", label: Implementation, format: markdown }
    charts:
      - { type: bar, label: Controls by status, groupBy: Status }   # shorthand; full chart spec in Section 6C
```

| Layout | Use |
| --- | --- |
| `fields` | Label and value pairs |
| `table` | Rows from a list of nodes, or from a `dataset`, with columns, sort, filter, group, and a detail pane |
| `dashboard` | Grid of chart and tile widgets over a dataset (Section 6C) |
| `list` | Simple bulleted list of values |
| `tree` | Outline of the source subtree (generic fallback) |
| `cards` | Repeated small field groups (for example, parties or components) |
| `text` | A single long Markdown or text field |

Section options: `id` (required, unique), `label`, `source` (FPath), `layout`, `collapsed` (default `false`), `formats` (limit to some formats), `emptyText`, `anchor` (enables comments on rows or the section), `sections` (nested sections).

A `dashboard` layout is also available; it arranges chart and tile widgets in a grid (Section 6C).

## 6A. Datasets and derived categories

Charts, tiles, and tables need the same records counted in the same way. A **dataset** names a set of records once, picks out the fields that matter, and can define **derived categories**: labels assigned by ordered, declarative rules. This is how a view expresses ideas like "internet-reachable and likely exploitable" without any code.

```yaml
datasets:
  - id: vulns
    label: Vulnerabilities
    source: "/vulnerability-report/findings/*"     # illustrative path
    key: id                                         # row key, also used as comment anchor
    fields:
      status:        { path: status }
      disposition:   { path: disposition }          # for example open, accepted, false-positive
      reachable:     { path: internet-reachable, type: boolean }
      exploitable:   { path: likely-exploitable, type: boolean }
      impact:        { path: impact-rating }         # for example N1 to N5
      severity:      { path: severity }
      detected:      { path: detected-date, type: date }
      dueDate:       { path: remediation-due, type: date }
    derived:
      exposure:                                     # first matching rule wins
        label: Exposure class
        rules:
          - when: { all: [ { field: reachable, equals: true }, { field: exploitable, equals: true } ] }
            value: IRV+LEV
          - when: { all: [ { field: reachable, equals: false }, { field: exploitable, equals: true } ] }
            value: NIRV+LEV
          - when: { field: exploitable, equals: false }
            value: NLEV
        otherwise: Unclassified
      overdue:
        rules:
          - when: { all: [ { field: dueDate, before: today }, { field: disposition, equals: open } ] }
            value: Overdue
        otherwise: On time
```

Condition operators (fixed list, no expressions): `equals`, `notEquals`, `in`, `notIn`, `exists`, `missing`, `before`, `after`, `between` (dates and numbers), `contains` (text, case-insensitive), combined with `all`, `any`, and `not`. Nesting depth is limited to 4. `today` is the only built-in relative value, plus offsets such as `today-30d`.

Field `type` values: `text` (default), `number`, `boolean`, `date`, `datetime`. Values that do not convert are counted as "Unparseable" instead of being dropped silently.

Mainly for CSV, a field can use `column: <header>` instead of `path`. Type conversion can be told how the file writes values: `formats` for dates (for example `["MM/DD/YYYY", "YYYY-MM-DD"]`, tried in order) and `true` and `false` lists for booleans (for example `true: [Yes, Y, TRUE]`). See [parsers-and-formats Section 9A](parsers-and-formats.md#9a-csv-as-a-machine-readable-input).

The field paths and category names above are illustrative. Real vulnerability report formats name their fields differently, and the published definitions of each category belong to whoever defines the format. That is why program-specific views ship as schema packs (Section 6D), not in the core.

## 6B. Pages

A schema can split a document into tabs. Each page holds sections, including dashboards. The page list is saved in the schema, so every document of that type opens with the same pages.

```yaml
pages:
  - id: overview
    label: Overview
    sections: [ { id: summary-dashboard, layout: dashboard, dataset: vulns, widgets: [ ... ] } ]
  - id: open
    label: Open findings
    sections: [ { id: open-table, layout: table, dataset: vulns, filter: { field: disposition, equals: open } } ]
  - id: accepted
    label: Accepted and false positive
    sections: [ { id: accepted-table, layout: table, dataset: vulns,
                  filter: { field: disposition, in: [accepted, false-positive] } } ]
```

Schemas are bound to document types through `schemaBindings` in the app config (see [app-config](app-config.md#3-settings-file)), so "the dashboard for this CSP's VDR folder" is a saved setting.

## 6C. Charts, tiles, and dashboards

```yaml
- id: summary-dashboard
  layout: dashboard
  dataset: vulns
  columns: 3                                   # grid columns; reflows to 1 column at high zoom
  widgets:
    - { type: tile, label: Total findings, measure: count }
    - { type: tile, label: Open, measure: count, filter: { field: disposition, equals: open } }
    - { type: tile, label: Overdue, measure: count, filter: { field: overdue, equals: Overdue }, tone: danger }
    - type: bar
      label: Findings by disposition
      groupBy: disposition
      measure: count
      order: [open, accepted, false-positive]
    - type: bar
      label: Open findings by exposure class
      filter: { field: disposition, equals: open }
      groupBy: exposure
      order: [IRV+LEV, NIRV+LEV, NLEV, Unclassified]
    - type: stacked-bar
      label: Open findings by impact rating and exposure
      filter: { field: disposition, equals: open }
      groupBy: impact
      stackBy: exposure
      order: [N5, N4, N3, N2, N1]
    - type: matrix
      label: Reachability by exploitability
      rows: reachable
      columns: exploitable
    - type: timeline
      label: Detected per month
      dateField: detected
      bucket: month
```

| Widget | Use |
| --- | --- |
| `tile` | A single number, with an optional comparison such as "of total" |
| `bar`, `stacked-bar` | Counts or sums per category, optionally stacked by a second category |
| `donut` | Share of a total. Limited to 6 slices; the rest are grouped as "Other" |
| `matrix` | Cross-tab of two categories, shown as a table with shaded cells |
| `timeline` | Counts per day, week, month, or quarter from a date field |
| `table` | Top N rows, for example the 10 oldest open findings |

Measures: `count`, `countDistinct: <field>`, `sum`, `min`, `max`, `average` (on numeric fields). Each widget may add its own `filter`, `groupBy`, `stackBy`, `order`, `limit`, and `buckets` (for numeric or date ranges).

Behavior:

- Charts are computed by the core over **every** record in the dataset, never only the rows on screen. Section 11 of the [parsers spec](parsers-and-formats.md#11-document-sessions-and-memory-model) explains how.
- Selecting a bar, slice, tile, or matrix cell opens the matching rows as a filtered table. The filter is shown as removable chips.
- Every chart has a "Show as table" toggle with the same numbers. Colors come from a fixed palette that is safe for color-vision deficiencies, and categories are also labeled directly.
- A document that lacks some fields still renders. Widgets whose fields are missing show "No data for this chart in this file" rather than failing the page.
- **Ad hoc charts**: any table column has "Chart this column," which builds a bar chart for the current filter without editing a schema. "Save to view" adds it to the user's copy of the schema.
- Charts count and group what is in the file. They do not compute scores, risk ratings, or compliance results.
- Export: copy the chart's numbers as CSV or Markdown. Exporting chart images is not planned for v1.

## 6D. Where program-specific dashboards live

The chart engine is generic and ships in the core. Dashboards tied to a specific program's vulnerability reporting format (for example, exposure classes or impact ratings defined by that program) ship as **schema packs**, so the core stays program-neutral (ASSUMPTION A-18). The repository includes an example pack, `packs/examples/example-publisher.vuln-report-dashboard`, built against a synthetic vulnerability report fixture. It shows a complete dashboard that partners can adapt to the real format (ASSUMPTION A-25: real field names are confirmed with whoever publishes that format).

Some programs publish their own **plain JSON schemas** (not OSCAL) for machine-readable submission artifacts, such as package overviews, vulnerability detail reports, or change notifications. Those are handled the same way: detection reads the file's `$schema` value, a pack view matches it with `jsonSchemaId`, and the pack can ship the schema file for the structure check. The repository includes a second example pack, `packs/examples/example-publisher.program-json-artifacts`, and a synthetic fixture whose `$schema` points at one such published schema, so detection of a non-OSCAL package is tested from the start ([issue #2](https://github.com/foliolight/foliolight/issues/2)).

The one program's schemas that ship with the app are the JSON schemas FedRAMP publishes. They are delivered as a **bundled pack** (`foliolight/fedramp-json`, built-in trust level), not as core code, and can be switched off. See [bundled-program-schemas](bundled-program-schemas.md). Their views use the real field names from the published schemas, which is why A-25 now applies only to category definitions the schemas do not spell out.

## 7. Formatters

Formatters come from a fixed list. Schemas cannot define new ones.

| Formatter | Behavior |
| --- | --- |
| `text` | Plain text (default) |
| `markdown` | Safe Markdown (no raw HTML, no remote images) |
| `date`, `datetime` | ISO 8601 shown in local format |
| `number`, `bytes`, `percent` | Numeric formatting |
| `boolean` | Yes or No |
| `enum-label` | Map raw values to labels via `enumLabels` |
| `badge` | Text plus an icon from a fixed palette (`success`, `warning`, `danger`, `info`, `neutral`). Never color alone. |
| `control-id` | Uppercase display and natural sort (AC-2 before AC-10) |
| `control-family` | Derives a family from a control ID (AC-2.1 becomes AC) |
| `uuid-link` | Resolves a UUID in the same document and links to it |
| `file-link` | Relative path shown as a safe launch link |
| `url` | Shown as text with a copy action, per link policy |
| `count` | Number of matched nodes |
| `list` | Joins multiple matched values |
| `code` | Monospace, no interpretation |
| `image` | Inline image from base64 or a library-local file, within limits |

## 8. Links and attachments

```yaml
links:
  remoteUrls: copy-only          # copy-only | confirm-open (still subject to app policy)
  localFiles: confirm-unknown    # always validated by the safe launcher
attachments:
  source: "/*/back-matter/resources/*"
  inlineImages: true
  maxInlineBytes: 5242880
```

App policy and the safe launcher always take precedence over schema settings. A schema can make link handling **stricter** but never looser than policy.

## 8A. Cross-document links

Review work crosses files: a POA&M item points at a control in the SSP, and an assessment result points at its plan. A cross-document link declares that a field holds an identifier found in **another primary file of the same CSP**, and selecting it opens that file at the match.

```yaml
links:
  crossDocument:
    - id: poam-to-ssp-control
      label: Open control in SSP
      field: "related-controls/*/control-id"     # FPath relative to the row or section
      target:
        documentType: SSP                        # same CSP, resolved through the library mapping
        version: current                         # current | same-period (closest version by date)
        idKind: oscal-control-id                 # oscal-control-id | oscal-uuid | schema-key | json-pointer
        schema: builtin:oscal/ssp                # optional: view to open the target with
```

Resolution:

1. Find the target document type in the same CSP folder. Use its current version unless `same-period` is set, in which case use the version whose date is closest to (and not after) the source file's date.
2. If that file already has a session, look up the identifier in its ID index.
3. Otherwise, read the target file once to build its ID index. This happens only when the user selects the link, never while listing or rendering. It shows progress if it takes longer than half a second.
4. Open the target in a new tab, scrolled to and highlighting the match. If it is not found, say so plainly ("AC-2(4) is not in the current SSP, ssp-2026-10.json") and offer to open the file anyway.

Known limits, stated deliberately:

- No joins. A view cannot show columns from two files side by side in one table, and charts cannot count across files.
- Same CSP only. Links never cross into another CSP's folder.
- Local files only. Identifiers that point at remote documents are shown as text.
- The target path goes through the same path validation as every other local link (see [threat model](../security/threat-model.md#63-paths-and-safe-external-launch)).

## 9. Extending another schema

`extends: builtin:oscal/ssp@^1` inherits a base schema. The extending schema can:

- Replace a section by using the same `id`.
- Add new sections, positioned with `before: <id>` or `after: <id>`.
- Remove a section with `{ id: <id>, remove: true }`.
- Add `hidden` patterns.
- Replace a dataset by using the same dataset `id`, while keeping the base schema's pages, charts, and tiles. This is how one dashboard serves the same kind of report delivered in different formats: a CSV view extends the JSON view, changes `match` to `formats: [csv]` with `csvHeaders`, and maps the same field names to CSV columns.

  ```yaml
  apiVersion: foliolight/view-schema/v1
  kind: ViewSchema
  metadata: { id: example-publisher/vuln-report-csv, version: 1.0.0, title: Vulnerability report (CSV) }
  extends: example-publisher/vuln-report-dashboard@^1
  match:
    formats: [csv]
    all: [ { csvHeaders: { includes: ["Finding ID", "Disposition"] } } ]
  datasets:
    - id: vulns                      # same id: replaces the base dataset, keeps all pages and charts
      source: /rows/*
      key: { column: Finding ID }
      fields:
        disposition: { column: Disposition }
        # ...same field names as the base view
  ```

  A replaced dataset must define every field name the base pages use. Validation reports any that are missing.

There is one level of inheritance only (no chains), to keep behavior predictable.

## 10. Precedence and selection

When several schemas match a file, the order is:

1. A user binding in app config (`schemaBindings`) for that library, CSP, or document type.
2. User-local schemas (created in the builder).
3. Pack schemas, by `priority` (higher wins), then by trust level (signed first), then by pack ID alphabetically.
4. Built-in schemas.
5. Generic fallback (Outline view).

The UI always shows which schema is in use and offers a **View as** chooser listing all matching schemas.

## 11. Inference algorithm (schema builder and `schema infer`)

Inference produces a draft that is useful immediately and easy to edit.

1. Parse up to the first 50,000 nodes of the sample (bounded).
2. Objects become `fields` sections, in document order.
3. Arrays of objects become `table` sections. Columns are scalar keys present in at least 30 percent of sampled items, ordered by frequency and then document order. Maximum 8 columns by default; the rest go to the detail pane.
4. Keys are humanized for labels (`implementation-status` becomes "Implementation status").
5. Fields named `uuid`, `id`, or ending in `-id` become `rowKey` and `anchor` candidates.
6. Value sniffing assigns formatters: ISO 8601 dates, URLs, UUIDs, long strings (collapsed `text` or `markdown`), base64 with a media type (`image` when it is an image type).
7. If the file is a recognized OSCAL model, the built-in schema is offered as the starting point instead.
8. The output YAML includes comments explaining each inferred choice.

## 12. Versioning

- `apiVersion` carries the major version of the spec. A new major version is required for breaking changes, and Foliolight supports the previous major version for at least one release cycle after a new one ships.
- Minor additions (new optional fields, new formatters) do not change `apiVersion`. Older app versions ignore unknown fields with a warning.
- Each schema's own `metadata.version` is semantic versioning, used for pack updates and `extends` ranges.

## 13. Validation

- `foliolight-cli schema validate <file>` and the builder both run JSON Schema validation, FPath syntax checks, unique section IDs, formatter names, and `extends` resolution.
- Errors report file, line, column, and a plain-language message.

## 14. Fit rules

Fit rules say what a file must contain for a view to work. They drive the fast fit check that runs every time a file is opened in a folder that already has a view chosen (see [opening-files](opening-files.md)).

```yaml
fit:
  required:                              # the view cannot work without these
    - path: /vulnerability-report/findings
      kind: array
    - path: "/vulnerability-report/findings/*/id"
  expected:                              # the view uses these; missing ones blank out columns, not the page
    - "/vulnerability-report/findings/*/severity"
    - "/vulnerability-report/findings/*/disposition"
  newFields:
    watch: "/vulnerability-report/findings/*"   # report keys here that the view does not show or hide
    minShare: 0.2                               # ignore keys present in fewer than 20 percent of sampled records
  sample: 500                                   # records sampled for the quick check
```

Defaults when `fit` is omitted: every dataset `source` and `key` path is required, every column and field path is expected, and new fields are watched under each dataset `source`. The schema builder writes an explicit `fit` block so reviewers can see and adjust it.

## 15. Structure schema (informational check)

A view may name the published schema the file is supposed to follow, so Foliolight can run the informational structure check described in [opening-files](opening-files.md#4-informational-structure-check).

```yaml
structureSchema:
  kind: json-schema                       # json-schema | xsd (OSCAL XML, Phase 7)
  file: schemas/vuln-report-1.2.schema.json   # shipped inside the app or the pack, never downloaded
  appliesWhen: { jsonSchemaId: "https://example.org/schemas/*/vuln-report*.json" }
```

Built-in OSCAL views reference bundled OSCAL schemas. The bundled FedRAMP pack's views reference the bundled copies of the published FedRAMP JSON schemas, including their Common Definitions file for `$ref` resolution. Pack views may ship their own schema files; JSON Schema files are an allowed pack file type (see [schema-pack](schema-pack.md#allowed-file-types)).

# Specification: Exporting Tables and Chart Data (CSV and XLSX)

Status: Draft v0.1. Added at owner direction on 2026-11-20, prompted by agency requests to take vulnerability report data into spreadsheets.

## 1. Goals

- Any table or list in Foliolight can be exported to CSV or Excel (`.xlsx`) **exactly as it is on screen**: the same columns, column order, labels, filters, and sort.
- Exports contain text and numbers only. No images, attachments, embedded files, formulas, macros, or active links.
- Exporting is cheap and never freezes the UI.
- Every export asks where to save it, starting in Downloads, with a suggested file name based on the source file, document type, and page (Section 5A).

Export does not change the read-only principle: it creates a new file where the user asks, and never touches package files.

## 2. What can be exported

| Source | What is exported |
| --- | --- |
| Table section in Document view | Every row matching the current filters (not just the visible page), in the current sort, with visible columns in on-screen order and on-screen labels. |
| Dashboard drill-down table | The rows behind the selected tile or bar, with the active filter chips. |
| Chart ("Export chart data") | The chart's table alternative: one row per category (and series), with counts and percentages. |
| Outline view selection | Not exported in v1. Exporting arbitrary trees to a flat sheet is ambiguous; use Source view and copy instead. |
| Comments | Already covered by [comments export](comments-store.md). |

**Two choices on the export dialog:**

- **Values:** "As shown" (formatted labels, for example `High` and `2026-10-14`), the default, or "Raw values" (the underlying values from the file, for example `high` and `2026-10-14T00:00:00Z`).
- **Columns:** "Visible columns" (default) or "All fields in this dataset," which adds hidden scalar fields as extra columns.

## 3. Computational load

Export is lightweight because the work is already done:

- The [dataset engine](parsers-and-formats.md#11-document-sessions-and-memory-model) already holds each table's records as compact columns, with filters and sort applied as index lists. Export walks those indexes and streams rows to the output file. The document is not re-parsed.
- Long text values that are stored only as offsets are read from the source file in order as the rows are written.
- Writing runs on the background worker with a progress bar and Cancel. A cancelled or failed export deletes its partial output file.
- Expected cost: well under a second for typical vulnerability reports (thousands of rows), and a few seconds for 200,000 rows, dominated by disk writes. Budgets are listed in the [test strategy](../testing/test-strategy.md#6-performance-budgets).
- Memory: CSV streams with constant memory. The XLSX writer is used in constant-memory (streaming) mode, so memory does not grow with row count.

## 4. Safety rules

These apply to both formats and cannot be turned off by view schemas or packs.

| Risk | Rule |
| --- | --- |
| Images and attachments | Never included. Image and attachment cells become the text `[image omitted]` or `[attachment omitted: name.pdf]`. Base64 content is never written. |
| Formula injection | CSV: values starting with `=`, `+`, `-`, `@`, tab, or carriage return are prefixed with a single quote, the same rule as comment export. XLSX: every cell is written as a typed string, number, or date, never as a formula, so nothing is evaluated. |
| Macros and active content | XLSX only, never `.xlsm`. No VBA, external data connections, pivot caches, or embedded objects. |
| Links | URLs are written as plain text, not clickable hyperlinks. |
| Spreadsheet limits | Excel allows 1,048,576 rows per sheet and 32,767 characters per cell. If an export would exceed them, the dialog says so before writing. Rows continue on additional sheets (`Findings (2)`), and over-long cells are truncated with a visible `[truncated]` marker and a count on the info sheet. |
| Encoding | CSV is UTF-8 with a byte order mark (so Excel detects UTF-8), RFC 4180 quoting, and CRLF line endings. |
| Where files go | Always asks, through the standard Windows Save As dialog (Section 5A). No temporary copies and no silent writes. |
| Sensitivity | The dialog shows a one-line reminder: "Exports contain package data. Store and share them the way you would the package itself." Downloads may be synced to OneDrive in some organizations, and the help text says so. |

## 5A. Choosing where to save, and the suggested file name

Every export asks the user where to save it. There is no "quick export" that writes without asking.

- **Dialog:** the standard Windows Save As dialog, with the format (`.xlsx` or `.csv`) preselected from the export dialog.
- **Starting folder:** the user's **Downloads** folder (the Windows Known Folder, which may be redirected by the organization), every time (ASSUMPTION A-30). The user can browse anywhere. The setting `export.startFolder: last-used` starts in the last folder used instead.
- **Package folders:** if the user picks a folder inside a library, Foliolight asks once more: "This folder is part of your package library. Saving here mixes exports in with package files and may sync them back to the provider. Save here anyway?"
- **Existing files:** Windows asks before replacing a file, as usual.

**Suggested file name:**

```
<original file name without extension> - <document type> - <page or table name>.<xlsx|csv>
```

Examples:

```
vuln-2026-10 - VulnReports - Open findings.xlsx
ssp-2026-10 - SSP - Control implementations.csv
poam-2026-09 - POAM - Overdue items (chart data).xlsx
```

- Chart data exports add `(chart data)`.
- If a filter is active that is not part of the page itself (for example, the reviewer clicked "IRV+LEV"), it is added in short form: `vuln-2026-10 - VulnReports - Open findings - IRV+LEV.xlsx`.
- The user can edit the name freely in the dialog.

**Windows file name rules** applied to the suggestion (the user's own edits are checked by Windows itself):

| Rule | Handling |
| --- | --- |
| Characters not allowed in Windows file names: `< > : " / \ \| ? *` and control characters (codes 0 to 31) | Replaced with `_` |
| Trailing spaces and trailing dots | Removed |
| Repeated spaces or underscores | Collapsed to one |
| Reserved device names (`CON`, `PRN`, `AUX`, `NUL`, `COM1` to `COM9`, `LPT1` to `LPT9`), including with an extension | Prefixed with `_` |
| Length | The name is kept to 150 characters including extension, well under the 255-character limit per name. If the chosen folder would push the full path past 259 characters on systems without long path support, the original file name portion is shortened in the middle (`vuln-2026-10-very-lo...-detail`) so the document type and page name stay readable. |
| Empty result | Falls back to `Foliolight export.<ext>` |
| Unicode | Kept as is (Windows supports it), except invalid characters, which are replaced with `_` |

## 5B. Provenance

So that a spreadsheet can be traced back to its source:

- **XLSX** includes a final sheet named **About this export** listing the source file name and path, SHA-256 of the source file, CSP and document type, view name and version, the active filters and sort in plain words, row count, export time (UTC), and Foliolight version, followed by the line: "Exported by Foliolight for review convenience. This is not an official record. The source file is authoritative."
- **CSV** cannot carry a second sheet, so the dialog offers "Also save an info file", which writes `<name>.about.txt` with the same content. It is on by default.

Exporting a file of a different version is not possible from the same table: the export always reflects the file currently open, and the info sheet names it.

## 6. CLI

```
foliolight-cli export <file> --view <schema-id> --dataset <id> [--filter <field>=<value>]... --format csv|xlsx --out <path>
```

Uses the same engine and safety rules as the app. See [cli](cli.md#export).

## 7. Dependencies

- CSV is written by the core with a small, well-tested CSV crate.
- XLSX needs a writer crate (candidate: `rust_xlsxwriter`, which supports constant-memory mode). Its license and maintenance must pass the dependency policy before adoption (ASSUMPTION A-28).

## 8. Accessibility

- Export is available from each table's toolbar and column menu, from the chart menu, and from the command palette, all keyboard reachable.
- Progress and completion are announced through the polite live region. Completion offers "Open folder"; it does not open the spreadsheet automatically.

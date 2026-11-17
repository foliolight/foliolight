# Specification: Package Library Convention

Status: Draft v0.1. Spec ID: `foliolight/library/v1`.

## 1. Purpose

The user points Foliolight at a root folder, and Foliolight discovers CSPs, document types, and files automatically. This spec defines the default folder convention, how to customize it with mapping rules, how the "current" version is chosen, and how scanning stays lazy and safe on large or networked libraries.

## 2. Terms

| Term | Meaning |
| --- | --- |
| Library | A root folder registered in Foliolight. A user may register several. |
| CSP folder | A first-level folder under the library root, one per cloud service provider. |
| Document-type folder | A folder under a CSP folder that maps to a document type such as SSP or POAM. |
| Document type | A logical category (SSP, SAP, SAR, POAM, Other, or a user-defined type). |
| Primary file | A file that represents the document itself (for example, `ssp.json`). |
| Attachment | A supporting file in the same folder (diagram, PDF, spreadsheet). |
| Version group | All primary files of one document type for one CSP, ordered by the version strategy. |
| Current file | The version group member that Foliolight treats as the latest. |

## 3. Default convention

```
Packages/                        <- library root
  Aster Example Cloud/           <- CSP folder (display name = folder name)
    SSP/                         <- document-type folder
      ssp-2026-10.json           <- primary file (current)
      ssp-2026-07.json           <- primary file (older version)
      boundary-diagram.png       <- attachment
      .foliolight-comments.yaml  <- comments sidecar (created only when commenting)
    SAP/
    SAR/
    POAM/
    Other/
  Brightwater Example Hosting/
    ...
```

Default document types and recognized folder names (case-insensitive, whitespace and punctuation normalized):

| Document type | Default folder names and aliases |
| --- | --- |
| `SSP` | `SSP`, `System Security Plan` |
| `SAP` | `SAP`, `Assessment Plan`, `Security Assessment Plan` |
| `SAR` | `SAR`, `Assessment Results`, `Security Assessment Report` |
| `POAM` | `POAM`, `POA&M`, `POA and M`, `Plan of Action and Milestones` |
| `Other` | `Other`, `Misc`, `Supporting` |

ASSUMPTION A-14: `SAP` is included in the default scaffold alongside the owner's example types.

Folders under a CSP that match no rule are shown as **Unmapped** with their files still listed, so nothing is hidden.

## 4. Mapping rules

Mapping rules let agencies keep their existing layout. Rules come from two places, merged in this order (later wins on conflicts):

1. Built-in defaults (Section 3).
2. An optional `foliolight-library.yaml` at the library root. Foliolight reads this file but never writes it, except through an explicit `foliolight-cli library init` command.
3. Per-library rules in the app config (`libraries[].rules`), which let a user customize a library they cannot write to.

### Example `foliolight-library.yaml`

```yaml
apiVersion: foliolight/library/v1
kind: Library
metadata:
  name: Agency packages
cspFolders:
  depth: 1                  # CSP folders are first-level folders (default)
  exclude: ["_archive", "Templates"]
  displayNames:
    "BW-Hosting": "Brightwater Example Hosting"   # ASSUMPTION A-13: folder name is the default
documentTypes:
  - type: SSP
    match:
      names: ["01 - System Security Plan", "SSP"]
  - type: SAR
    match:
      glob: ["Assessment/*/Results", "SAR*"]       # path relative to the CSP folder
  - type: POAM
    match:
      regex: "^(poa&?m|plan of action).*$"
  - type: ConMon                                     # user-defined type
    label: Continuous monitoring
    match:
      names: ["ConMon", "Monthly"]
primaryFiles:
  include: ["*.json", "*.xml", "*.yaml", "*.yml"]
  exclude: ["*-schema.json"]
versioning:
  strategy: filename-pattern                         # mtime | filename-pattern | oscal-last-modified
  pattern: "(?P<date>\\d{4}-\\d{2}(-\\d{2})?)"
  compare: date                                      # date | semver | integer | natural
  fallback: mtime
```

### Rule semantics

- `names`: exact match after normalization (case-folded, trimmed, runs of spaces, dashes, and underscores collapsed).
- `glob`: matched against the path relative to the CSP folder, using `/` separators. `*` matches one segment, `**` matches any number.
- `regex`: matched against the normalized folder name. Rust `regex` syntax, which has guaranteed linear-time matching (no catastrophic backtracking). Maximum pattern length 512 characters.
- The first matching rule in file order wins. `foliolight-cli library verify` reports folders matched by more than one rule as warnings.
- Nested folders inside a document-type folder inherit its type. A file is listed under the nearest mapped ancestor.

## 5. Version and current-file rules

**Why not just "newest modified time":** file modified times change when files are downloaded, copied across drives or shares, restored from backup, or synced. The newest timestamp is not always the newest package. Strategies:

| Strategy | How it decides | Cost |
| --- | --- | --- |
| `mtime` (default) | Latest last-modified time. | Free (from directory listing). |
| `filename-pattern` | Extracts a version or date from the file name with a regex and compares it. | Free. |
| `oscal-last-modified` | Reads `metadata/last-modified` from the OSCAL header. | Reads up to the first 64 KB of each candidate, **only** when the user opens that document type, never during the initial listing. |

Rules:

- If the chosen strategy cannot rank a file (no pattern match, no OSCAL metadata), the `fallback` strategy is used, and the UI marks that file's ordering as "estimated."
- The user can **pin** any file as current. Pins are stored in app config (not in the library) keyed by library ID, CSP, and document type.
- Older versions are listed under the current file, newest first.

## 6. Lazy scanning

Listing never parses file contents.

| Behavior | Detail |
| --- | --- |
| Initial scan | Enumerates CSP folders and their direct children only. Document-type folders are enumerated when expanded, or in a background pass for local disks. |
| Metadata | Name, extension, size, last-modified time, and attributes from the directory enumeration itself. No per-file open. |
| Format hint | From extension only during listing. Content sniffing (first 64 KB) happens only when the user selects a file. |
| Streaming | Results stream to the UI in batches of 200 entries so large folders render progressively. |
| Limits | Default maximum depth below the library root: 8. Default maximum entries per folder before paging: 5,000. Both configurable. |
| Excluded entries | Hidden and system files, `desktop.ini`, `Thumbs.db`, Office lock files (`~$*`), temporary files (`*.tmp`, `*.partial`, `*.crdownload`), and Foliolight sidecars. |
| Caching | Listing results are cached in memory only, for the session. Nothing is written to disk. |

## 7. Symlinks, junctions, and reparse points

- Symbolic links, junctions, and other reparse points are **not followed** by default and are shown with an indicator and the reason.
- A per-library setting `followLinks: within-root` allows following links whose canonical target remains inside the library root. Links pointing outside the root are never followed.
- Loop protection: the scanner tracks visited volume and file IDs and stops on repeats.

## 8. OneDrive and cloud-only placeholders

Files that are cloud-only placeholders (Files On-Demand) have attributes such as `FILE_ATTRIBUTE_RECALL_ON_DATA_ACCESS`. Reading them causes the sync client, not Foliolight, to download them.

- They are listed with a "cloud-only" indicator.
- Content sniffing is skipped for them.
- Opening one shows a notice: "This file is stored in the cloud. Opening it will cause your sync client to download it." The user confirms once per session.

## 9. Network shares (SMB)

- Enumeration is per folder and lazy, so a slow share only slows the folder being expanded.
- File change notifications are not relied on for network paths. Refresh happens manually (F5 or the refresh button) and when the window regains focus (throttled to once per 30 seconds per library).
- For local disks, an optional file watcher may refresh listings. It runs only while the app is open.
- Timeouts: an enumeration that exceeds 10 seconds shows a "still working" state with a cancel option; it never freezes the UI.

## 10. Opening files read-only

- Files are opened for read access with sharing that allows other processes to read, write, and delete (`FILE_SHARE_READ | FILE_SHARE_WRITE | FILE_SHARE_DELETE`), so CSP sync tools and editors are never blocked by Foliolight.
- If a file changes while open, Foliolight detects the size or modified-time change, shows "This file changed on disk," and offers reload. It never writes to the file.

## 11. Scaffolding

`foliolight-cli csp scaffold` (and the equivalent UI action) creates empty CSP and document-type folders. This is the only operation besides comments that writes inside a library, and it runs only on explicit user command. It never overwrites or removes existing folders or files.

## 12. Validation

`foliolight-cli library verify` reports:

- Root missing or unreadable.
- Unmapped folders and folders matched by more than one rule.
- Files over size limits, unreadable files, and skipped reparse points.
- Version groups where ordering is "estimated."
- Comment sidecars present, with comment and orphan counts.
- Unknown file types.

## 13. Open items

- Whether to support more than one library root per "workspace" view (for example, combining a local and a share library in one tree). Default: separate libraries, switchable from the toolbar.

# Specification: User Interface and Accessibility

Status: Draft v0.1. Wireframes are text sketches that show layout and content, not visual design.

These wireframes are validated with a clickable prototype and reviewer sessions before UI build-out; see [ui-prototyping](../design/ui-prototyping.md).

## 1. Design goals

- A non-technical reviewer gets value within a minute of pointing Foliolight at a folder.
- Consistent layout for every CSP and every format, so switching between 30 packages feels the same.
- Every function works by keyboard and with a screen reader.
- Calm, unobtrusive status: no blocking dialogs for background states (updates, indexing).

## 2. Main window

```
+--------------------------------------------------------------------------------------+
| [=] Foliolight  | Library: [Agency packages  v] | [Find in file...        ] | (o) [?] [*] |
+----------------------+---------------------------------------------+-----------------+
| LIBRARY         [R]  | Aster Example Cloud > SSP > ssp-2026-10.json | DETAILS|COMMENTS|
| [Filter CSPs...   ]  | Current | 12.4 MB      [ssp-2026-10 ..41ab] |                 |
|                      | [Document] [Outline] [Source]   View as: [v]| (panel content) |
| v Aster Example Cl.  |---------------------------------------------|                 |
|   v SSP          (2) |                                             |                 |
|     * ssp-2026-10    |              (viewer content)               |                 |
|       ssp-2026-07    |                                             |                 |
|   > SAP          (1) |                                             |                 |
|   > SAR          (3) |                                             |                 |
|   > POAM         (6) |                                             |                 |
|   > Other        (4) |                                             |                 |
| > Brightwater Exam.  |                                             |                 |
| > Cobalt Example S.  |                                             |                 |
+----------------------+---------------------------------------------+-----------------+
| JSON | OSCAL SSP 1.1.2 | View: builtin:oscal/ssp | Indexed | 2 comments (1 changed)  |
+--------------------------------------------------------------------------------------+
 Legend: [R] refresh  (o) update status icon  [?] help  [*] settings  * current version
         [ssp-2026-10 ..41ab] file identity badge (file name and end of SHA-256), see Section 4A
```

Regions (each is an ARIA landmark):

| Region | Landmark | Content |
| --- | --- | --- |
| Toolbar | `banner` | Library switcher, find, update status, help, settings |
| Library | `navigation` named "Library" | CSP and document tree |
| Viewer | `main` | Breadcrumb, file header, view tabs, content |
| Side panel | `complementary` named "Details and comments" | Details or Comments tab |
| Status bar | `contentinfo` | Format, model, schema, indexing state, comment summary |

Panels are resizable with the mouse and with keyboard (focus the splitter, then arrow keys). Both side regions can be collapsed.

## 3. Library browser

```
LIBRARY                                      [R]
[Filter CSPs and files...                     ]
Sort: [Name v]   Show: [x] Older versions  [ ] Unmapped

v Aster Example Cloud                    3 types
  v SSP                                  2 files
    * ssp-2026-10.json    JSON  12.4 MB  2026-10-14   [current]
      ssp-2026-07.json    JSON  11.9 MB  2026-07-02
    boundary.png          PNG    240 KB  2026-10-14   [attachment]
  > SAR                                  3 files
  > POAM                                 6 files
v Brightwater Example Hosting           mapped by rules
  > SSP  (from "01 - System Security Plan")
  > Unmapped: "Misc 2"                   [map...]
v Cobalt Example Systems
  v POAM
    ! poam-full.xml       XML    610 MB  2026-09-30   [over size limit]
    @ poam-link           link           skipped: points outside library
    ~ poam-2026-08.yaml   YAML    2 MB   cloud-only
```

- Rows show only listing metadata. Nothing is parsed until the user opens a file.
- Version ordering marked "estimated" when the strategy fell back (see [library-convention](library-convention.md#5-version-and-current-file-rules)).
- Context menu (also Shift+F10): Open, Open in new tab, Pin as current, Show in Explorer, Copy path, Copy SHA-256, Map folder to type, Choose view for this folder.
- When the files in a folder are ordered by the date inside each file and that disagrees with the dates on disk, the folder shows a small note explaining which order is used.
- Status markers (`*`, `!`, `@`, `~`) always come with text, so meaning never depends on a symbol or color alone.

## 4. File viewer

### Document view (schema-driven)

```
Aster Example Cloud > SSP > ssp-2026-10.json
[Document] [Outline] [Source]                         View as: [builtin:oscal/ssp v]
Jump to: [Overview] [Parties] [Components] [Controls] [Back matter]
-------------------------------------------------------------------------------
v System overview
    System name     Aster Example Cloud Platform
    Sensitivity     Moderate
    Description     The platform provides ...                     [more]

v Control implementations (325)                 [Chart: by status]
  Filter: Family [All v]  Status [All v]  Text [            ]   Group by: [Family v]
  +----------+-------------+------------+--------------------------------+
  | Control  | Status      | Statements | Notes                          |
  +----------+-------------+------------+--------------------------------+
  | AC-1     | Implemented | 2          |                                |
  | AC-2     | Partial     | 5          | 2 comments (1 content changed) |
  | AC-2(1)  | Planned     | 1          |                                |
  +----------+-------------+------------+--------------------------------+
  Rows 1 to 50 of 325                          [Export...]    [Load more]

> Back matter (12 resources)
    boundary-diagram  image/png  (inline preview)
    policy-document   ./attachments/policy.pdf   [Open in default app...]
    vendor-site       https://example.com        [Copy link]
```

### Dashboard page (schema-driven)

Pages defined by the view schema appear as tabs inside the Document view. Example: a vulnerability report with the example dashboard pack.

```
Cobalt Example Systems > VulnReports > vuln-2026-10.json
[Document] [Outline] [Source]          View as: [example-publisher/vuln-report-dashboard v]
Pages: [Overview] [Open findings] [Accepted and false positive] [All findings]
-------------------------------------------------------------------------------------
Filters: (none)                                              [Edit charts] [Show all as tables]
+-------------------+ +-------------------+ +-------------------+
| Total findings    | | Open              | | Overdue (!)       |
|   1,284           | |   412  (32%)      | |   37              |
+-------------------+ +-------------------+ +-------------------+
Findings by disposition                    Open findings by exposure class
Open            ######################  412   IRV+LEV       ######        58
Accepted        ##############          268   NIRV+LEV      ##########   121
False positive  ######                  104   NLEV          ############ 226
Fixed           ####################### 500   Unclassified  #              7
                         [Show as table]                           [Show as table]
Open findings by impact rating, stacked by exposure class
N5  ##=                 9      Legend: # IRV+LEV  = NIRV+LEV  . NLEV
N4  ###===....         31
N3  ####======.......  88                                         [Show as table]
```

- Selecting "IRV+LEV" opens the **Open findings** page filtered to that class. The filter shows as a chip ("Exposure class: IRV+LEV  x") that can be removed by keyboard.
- **Edit charts** opens the dashboard editor from the schema builder for this page, with live preview. Changes save to the user's copy of the schema for this document type.
- **Chart this column** is available in every table column's header menu.
- Charts that need fields this file does not have show "No data for this chart in this file."
- Every chart menu has **Export chart data**, and every drill-down table has **Export...**, which writes exactly what is on screen to CSV or Excel (see [export](export.md)).

Chart accessibility:

- Each chart is an SVG with `role="img"` and an accessible name and summary (for example, "Bar chart, open findings by exposure class, 4 categories, largest NLEV 226"), followed by a focusable list of its bars. Each bar announces its label, value, and percentage, and Enter drills down.
- "Show as table" is always available and remembered per user.
- Patterns or direct labels accompany colors; the palette is safe for color-vision deficiencies and adapts to Windows high contrast themes.

### Outline view (any structured file)

```
[Expand all] [Collapse all]   Find: [implementation-status   ] 14 matches  [<] [>]
v system-security-plan  {object, 6 keys}
  > uuid  "8a3f..."
  v metadata  {object, 7 keys}
      title  "Aster Example Cloud SSP"
      last-modified  2026-10-14T10:00:00Z
  > system-characteristics  {object}
  v control-implementation  {object}
    v implemented-requirements  [array, 325 items]
      > [0]  control-id: ac-1
      > [1]  control-id: ac-2      (2 comments)
  ...
Path: /system-security-plan/control-implementation/implemented-requirements/1  [Copy]
```

### Source view

Read-only text with line numbers and syntax highlighting, paged by visible window. Go to line (Ctrl+G). Selection of a line range can start a line-range comment.

### Large-file states

```
Opening poam-big.json (310 MB)
[==========>                   ] 34% indexed   [Cancel]
You can browse the top-level sections while indexing continues.
```

Progress is announced through a polite live region at 25 percent steps, not every update.

### 4A. File identity badge and hash check

The top-right corner of the viewer header always shows which exact file is open: the file name (shortened in the middle if needed) and the **last 8 characters of its SHA-256 hash**. This answers "which version am I looking at?" at a glance and makes it quick to compare with a hash list from the CSP ([issue #4](https://github.com/foliolight/foliolight/issues/4)).

- The hash is computed on the background worker during the same read that indexes the file, so it adds almost no cost. Until it is ready, the badge reads `ssp-2026-10 ..computing`.
- The accessible name is the full sentence: "Open file ssp-2026-10.json, SHA-256 ending 41ab 9c03. Select to check the hash."

Selecting the badge (or pressing Ctrl+Shift+H) opens a small popover, not a blocking dialog:

```
ssp-2026-10.json                                              [x]
Aster Example Cloud > SSP          12.4 MB   Modified 2026-10-14
SHA-256  3b7e 0c55 ... 9d12 41ab 9c03                     [Copy full hash]

Compare with a hash from the provider
[ paste a hash here                                              ]
  Result: Match. This file's SHA-256 matches the hash you pasted.

Hash list in this folder: SHA256SUMS lists this file. Match.
```

Behavior:

- **Paste box.** Accepts uppercase or lowercase, spaces or line breaks inside the hash, a `sha256:` prefix, or a whole line copied from a `sha256sum` style list. It compares as soon as something is pasted. No button is needed.
- **Results**, each with an icon and text, never color alone:
  - "Match. This file's SHA-256 matches the hash you pasted."
  - "Does not match. This is not the file the hash was made from. It may be a different version, or it may have been changed. Check with the provider before relying on it."
  - "This looks like a SHA-384 (or SHA-512) hash." Foliolight computes that algorithm on request and compares.
  - "This looks like an MD5 (or SHA-1) hash. Those are too weak to rely on for this check. Ask the provider for SHA-256." The app does not compute them (ASSUMPTION A-27).
  - "That doesn't look like a hash." Shown for anything else.
- **Hash list in the folder.** If the document-type folder contains a hash list (`SHA256SUMS`, `*.sha256`, or `checksums.txt` in `sha256sum` format), the popover shows whether it lists this file and whether it matches. A hash list in the same folder only shows the file is unchanged since the list was made. It does not prove who made either one, and the popover says so in its help text.
- **Narrow wording.** The UI never says "authentic," "verified," or "genuine." A match only means the bytes are identical to what the hash describes.
- Results are kept for the session only. They are not saved, so a later check always reflects the file as it is now.
- If the file changes on disk while open, the badge shows "File changed on disk" and the hash is recomputed after reload.

### 4B. Notices when a file opens

Whether the folder's view fits this file, suggested views for a new kind of file, and plain-language error panels are specified in [opening-files](opening-files.md). They appear as a notice region between the view tabs and the content, never as blocking dialogs.

## 5. Schema builder

A guided, five-step flow with a live preview. Every step is a normal page, not a modal, so screen reader users can move freely.

```
Schema builder: vendor-export.json                       Step 2 of 5: Choose content
[1 Sample] [2 Content] [3 Layout] [4 Details] [5 Save]
-----------------------------------------------------------------------------------
STRUCTURE (inferred)                      | PREVIEW
[x] export                                |  Vendor export
  [x] summary        -> Fields            |  ---------------------------------
  [x] findings       -> Table (12 cols)   |  Summary
      [x] id         Column 1  (row key)  |    Generated   2026-10-01
      [x] severity   Column 2  Badge      |    Source      Example Scanner
      [x] title      Column 3             |  Findings (48)
      [ ] raw_blob   Hidden (long text)   |    ID     Severity  Title
  [ ] debug          Hidden               |    F-001  High      TLS ...
                                          |
[Back]                                      [Next: Layout]
```

| Step | What the user does |
| --- | --- |
| 1 Sample | Pick a sample file. Foliolight infers a draft or offers a matching built-in schema as a starting point. |
| 2 Content | Include or hide nodes. Choose section type: fields, table, list, cards, text. |
| 3 Layout | Reorder sections and pages, rename labels, choose table columns, sorting, grouping, and filters. Define datasets and derived categories with a rule editor (pick a field, pick an operator, pick a value). Build dashboard pages by adding tiles and charts from a list; each widget previews with real counts from the sample. |
| 4 Details | Formatters, enum labels, badges, anchors for comments, link and attachment handling (including links to other documents from the same CSP), match rules (which files this applies to), and fit rules shown in plain words ("Files must have: a list of findings, each with an ID"). |
| 5 Save | Name and ID, validation results with plain-language messages, save to user schemas, or export as a file for a pack. A "Show YAML" toggle shows the exact file. |

Sensible defaults: if the user clicks Next through every step, the result is still a usable view.

## 6. Comments panel

```
DETAILS | COMMENTS (5)                                [+ Add comment]
Filter: [Open v] [All severities v]      Export: [Markdown] [CSV] [JSON]
-----------------------------------------------------------------------
On this item: AC-2
  [Open] [Moderate]  J. Reviewer  2026-08-03
  Clarify the frequency of privileged account reviews.
  ! Content changed since this comment (ssp-2026-07 -> ssp-2026-10)
    Then: "Account management is implemented through the central..."
    [Confirm still applies] [Resolve] [Edit]

In this file (3)
  AC-6(9)   [Open]      Logging scope unclear                 [Go to]
  CM-8      [Resolved]  Inventory matches attachment          [Go to]
  File      [Info]      Overall: well organized               

Orphaned (1)
  SC-7(4)   [Open]  "Boundary protection for ..." not found in this version
            [Re-anchor...] [Make file-level] [Resolve] [Delete]
-----------------------------------------------------------------------
Comments are saved in this folder (.foliolight-comments.yaml).  [Learn more]
```

Read-only folder state:

```
COMMENTS
Commenting is turned off for this folder because Foliolight cannot write here.
Reason: Access denied on \\fileserver\ato\Packages\Aster Example Cloud\SSP
Ask your administrator for write access to this folder, or copy the package
to a folder you can write to. Existing comments (if any) are shown read-only.
```

## 7. Update status indicator

A small icon in the toolbar. Its tooltip is also its accessible name, so screen readers hear the same text. Clicking opens a small popover, never a blocking dialog.

| State | Icon (shape) | Tooltip and accessible name |
| --- | --- | --- |
| Disabled | Circle with slash | "Update checks are off. Turn on in Settings." (or "...managed by your organization.") |
| Not yet checked | Hollow circle | "Update check has not run yet." |
| Checking | Rotating arrows (static if reduced motion) | "Checking for updates..." |
| Up to date | Check mark | "Foliolight 1.2.0 is up to date. Checked today at 9:02." |
| Update available | Up arrow with dot | "Foliolight 1.3.0 is available. Select for details." |
| Could not check | Cloud with question mark | "Could not check for updates (no network or blocked). Foliolight works normally. Last successful check: 2026-11-02." |
| Downloading | Down arrow with progress ring | "Downloading 1.3.0: 40 percent." |
| Verified | Shield with check | "1.3.0 downloaded and verified. Select to open folder." |
| Verification failed | Shield with exclamation | "Download failed verification and was deleted. Do not use it." |

Popover for "Update available":

```
Foliolight 1.3.0 is available (you have 1.2.0)
Released 2026-11-10. Release notes: [Copy link]
[Download and verify...]   [Remind me later]   [Turn off update checks]
```

## 8. Keyboard map

| Keys | Action |
| --- | --- |
| Ctrl+O | Choose library folder |
| Ctrl+F | Find in current file |
| Ctrl+Shift+F | Filter library |
| F6 / Shift+F6 | Move between regions (toolbar, library, viewer, side panel, status bar) |
| Alt+1, Alt+2, Alt+3 | Document, Outline, Source view |
| Arrow keys | Move in trees and grids |
| Right / Left | Expand or collapse tree node |
| Home / End | First or last item |
| \* (numpad) | Expand all siblings (tree pattern) |
| Enter | Open item or activate |
| Shift+F10 or Menu key | Context menu |
| Ctrl+Alt+M | Add comment on the focused item |
| Ctrl+G | Go to line (Source) or go to ID (Document, Outline) |
| Ctrl+Shift+H | Open the file identity and hash check popover |
| Ctrl+Shift+E | Export the focused table or chart data |
| F5 | Refresh library |
| Ctrl+Plus / Ctrl+Minus / Ctrl+0 | Zoom in, out, reset |
| Esc | Close popover or cancel operation |

All shortcuts are listed in Help and can be discovered through tooltips.

## 9. Accessibility requirements (WCAG 2.1 AA and Section 508)

### Structure and semantics

- Landmarks for every region; one `h1` per view; heading levels follow the section hierarchy so screen reader users can jump by heading.
- Library and Outline use the ARIA **treeview** pattern with roving `tabindex`. Each item sets `aria-level`, `aria-setsize`, `aria-posinset`, and `aria-expanded`, so virtualized trees still announce "item 12 of 325, level 3."
- Tables use the ARIA **grid** pattern (or native tables for short static tables) with `aria-rowcount` and `aria-rowindex` for virtualized rows, and sortable headers with `aria-sort`.
- Tabs, dialogs, menus, and popovers follow the ARIA Authoring Practices patterns, with focus trapping only in true modal dialogs.

### Virtualization and screen readers

Virtualized content is only partly in the DOM, so screen reader "browse mode" cannot read off-screen items. Mitigations:

- Position attributes as above.
- "Jump to" section links and Go to ID.
- Table pages of 50 rows with an explicit "Load more" button rather than infinite scroll.
- An "Accessible full list" option for any table under 5,000 rows that renders every row without virtualization.

### Visual

- Text contrast at least 4.5:1, large text and UI components at least 3:1, in light and dark themes.
- Windows high contrast themes supported through the CSS `forced-colors` media query; system colors used, no information carried by background images.
- No information conveyed by color alone (badges have text and icon shape).
- Works at 200 percent zoom and at 400 percent with reflow for the viewer (panels stack vertically). Honors Windows text scaling.
- Visible focus indicator on every interactive element, at least 2 CSS pixels and 3:1 contrast.
- Respects reduced motion settings.

### Interaction

- Everything operable by keyboard with no keyboard traps.
- No time limits. Long operations are cancellable.
- Errors identified in text with suggestions (for example, schema validation messages with line and column).
- Live regions: polite for progress and background states; assertive only for errors that stop an action the user just started.
- Target size at least 24 by 24 CSS pixels for pointer targets where practical.

### Testing and reporting

| Check | When |
| --- | --- |
| axe-core automated checks on every component story and end-to-end page | Every pull request |
| Keyboard-only walkthrough script | Every release candidate |
| Narrator (Windows built-in), NVDA (latest), JAWS (latest available to testers) with Edge WebView2 | Every minor release |
| Windows high contrast themes and 200 and 400 percent zoom | Every minor release |
| Accessibility Conformance Report (VPAT 2.x, Section 508 edition) | Published at 1.0 and updated each minor release |

Known risk: JAWS licenses are not free. ASSUMPTION A-23: partners or agencies will provide JAWS testing; otherwise the ACR documents NVDA and Narrator results only.

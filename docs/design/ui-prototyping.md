# UI Prototyping and Feedback Plan

Status: Draft v0.1.

## 1. Purpose

Get agency reviewers and CSP partners to react to the interface **before** it is built, when changes are cheap. The [text wireframes](../specs/ui.md) are the starting point. A clickable prototype makes them concrete enough to test.

## 2. Tool choice

| Option | Pros | Cons |
| --- | --- | --- |
| **Penpot** (recommended default) | Open source (MPL-2.0), can be self-hosted, free, works in a browser, exports SVG and CSS. Fits an open-source, vendor-neutral project. | Smaller community than Figma; fewer ready-made components. |
| Figma (free or education tier) | The most familiar tool for designers and many partners; strong prototyping and commenting. | Proprietary and cloud-hosted; every reviewer needs an account; files live on Figma's servers. |
| Clickable HTML prototype (Storybook with the real components) | Tests real accessibility behavior with screen readers; becomes the living component reference. | More effort; better suited to the second round. |

A design tool is a working tool, not a shipped dependency, so the dependency license policy does not apply to it. Exported design files committed to the repository fall under the repository license (Apache-2.0) through `REUSE.toml`.

**Recommended approach:** round 1 in Penpot or Figma (owner or community choice, see the [open questions](../community/open-questions.md#10-additional-questions-raised-during-planning)). Round 2 as a Storybook prototype built from the first real components, so screen reader testing uses real markup.

## 3. Rules

- **Synthetic data only.** Use the fictional CSPs from the [test strategy](../testing/test-strategy.md). Never paste real package content into a cloud design tool.
- Design tokens (colors, type scale, spacing, focus styles) are defined once in the prototype and checked for WCAG 2.1 AA contrast before round 1.
- Screens are annotated for accessibility: heading levels, landmark regions, focus order, and accessible names for icon buttons.
- No government marks or employer branding in any mockup.

## 4. Screens in scope

1. First run and choosing a library.
2. Library browser with three CSPs, versions, unmapped folders, and warning states.
3. Document view for an SSP (sections, control table, filters).
4. Dashboard page for a vulnerability report: tiles, charts by disposition, exposure class, and impact rating, and drill-down to a filtered table.
5. Outline and Source views, including a large-file indexing state.
6. Schema builder: all five steps, including the chart and dashboard editor.
7. Comments panel: attached, changed, orphaned, and read-only folder states.
8. Update status indicator in each state, plus settings and policy-locked settings.
9. High contrast and 200 percent zoom variants of screens 2, 3, and 4.

## 5. Feedback rounds

| Round | Fidelity | Participants | Method |
| --- | --- | --- | --- |
| 1 | Clickable mockups | 3 to 5 agency reviewers and 2 to 3 CSP or partner staff | Moderated 45-minute sessions using a task script, plus asynchronous comments in the design tool |
| 2 | Storybook prototype with real components | Same mix, at least one screen reader user | Moderated sessions; keyboard-only and screen reader tasks |

Sample tasks:

- "Open the newest SSP for Brightwater Example Hosting and find the status of AC-2."
- "How many open findings for Cobalt Example Systems are both internet-reachable and likely exploitable? Show me the list."
- "Compare the July and October SSPs. Which of your comments no longer match the content?"
- "Create a view for this unfamiliar JSON file that shows findings in a table with a chart by severity."

Measures: task completion without help, time on task, System Usability Scale questionnaire after each session, and a list of issues rated by severity.

## 6. Outputs

- Findings filed as GitHub issues labeled `design-feedback`.
- Exported screens and annotations under `docs/design/` (PNG plus a short Markdown summary for each round).
- Updated [ui spec](../specs/ui.md) wireframes and design tokens.

## 7. Exit criteria (Phase 0B)

- At least 4 of 5 participants complete each core task without help in round 1, or the design is revised and retested.
- Design tokens approved and contrast-checked.
- No open design issue rated "blocks a core task."
- Round 2 (Storybook) is scheduled to run alongside Phase 1 UI build-out, not before it, so it does not delay the beta.

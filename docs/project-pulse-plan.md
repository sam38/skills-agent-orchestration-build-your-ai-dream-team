# Project Pulse Dashboard — Implementation Plan

## 1. Overview

Project Pulse is a small static web dashboard that helps Mona's team quickly
understand the health of active projects at a glance. It answers, for each
project: who owns it, what its current status is, what happened recently,
and how urgent/risky it is. The dashboard is a static app (no backend, no
build step) made of three files served from `app/`:

- `app/index.html` — page structure and markup for the dashboard shell and
  project cards.
- `app/styles.css` — visual design system (palette, typography, spacing,
  responsive layout, card/badge states).
- `app/project-data.json` — the data source: a top-level `projects` array,
  each entry containing `name`, `owner`, `status`, `recentActivity`, and
  `priority`.

A `.vscode/launch.json` configuration named **"Run Project Pulse Dashboard"**
lets a learner start a local static server rooted at `app/` and open
`index.html` directly, so the dashboard UI appears instead of a directory
listing.

## 2. File assignments

| File                     | Owner     | Notes |
|--------------------------|-----------|-------|
| `app/index.html`         | Coder (primary), Designer (secondary, structural review) | Coder writes semantic HTML skeleton and JS data-wiring. Designer reviews/adjusts markup structure and class hooks (`.dashboard`, `.project-card`) needed for styling and information hierarchy, but does not own final authorship. |
| `app/styles.css`         | Designer  | Full ownership: color palette, typography, spacing scale, responsive breakpoints, card/badge/priority visual states, `.dashboard` and `.project-card` selectors. |
| `app/project-data.json`  | Coder     | Full ownership: schema definition and sample data (`projects` array; `name`, `owner`, `status`, `recentActivity`, `priority` per entry). |
| `.vscode/launch.json`    | Coder     | Full ownership: strict JSON, no comments, `cwd` set to `${workspaceFolder}/app`, serves `app/` and opens `index.html`, launch config named "Run Project Pulse Dashboard". Designer must not modify this file. |

**Joint-touch rule for `app/index.html`:** Coder writes the initial HTML
skeleton and IDs/classes needed for data binding first. Designer then edits
the same file (in a separate, sequential phase) to refine layout structure,
accessibility markup (headings, landmarks, `aria-*`), and the exact class
hooks CSS will target. Both must coordinate — this file is never edited by
both roles in the same phase to avoid merge conflicts.

## 3. Designer responsibilities

- Define the visual design system in `app/styles.css`:
  - Color palette (background, surface, text, accent, status colors for
    e.g. on-track/at-risk/blocked, priority colors for high/medium/low).
  - Typography scale (heading sizes, body text, font stack, line-height).
  - Spacing scale/rhythm for consistent padding/margins across cards and
    sections.
  - Responsive breakpoints so the dashboard reflows from multi-column card
    grid (desktop) to single column (mobile).
  - Component states: `.project-card` default/hover states, status badges
    per status value, priority indicator treatment (e.g., color-coded
    left border, icon, or pill).
  - Visual polish: rounded corners (`border-radius`), elevation
    (`box-shadow`), sufficient contrast, clear focus states.
- Review/refine the structural layout of `app/index.html` (after Coder's
  initial skeleton exists): information hierarchy (dashboard title →
  project grid → individual card contents), semantic HTML and accessible
  landmarks, and the exact class names (`.dashboard`, `.project-card`,
  status/priority classes) the CSS will hook into.
- Do not edit `app/project-data.json` or `.vscode/launch.json`.
- Report design decisions, files touched, and how to visually verify the
  result (e.g., "resize to 375px width and confirm cards stack").

## 4. Coder responsibilities

- Build the initial semantic HTML skeleton in `app/index.html`: a page
  shell, a `.dashboard` container, and a template/loop structure (via JS)
  that renders one `.project-card` per project.
- Define the `app/project-data.json` schema and sample data: top-level
  `projects` array; each object has `name`, `owner`, `status`,
  `recentActivity`, `priority` (and optionally a short summary field per
  the brief's "short contributor-friendly summary" requirement — confirm
  with Orchestrator/brief before adding extra fields, since the brief's
  required field list is fixed to those five).
- Write the JS logic (inline `<script>` in `index.html` or a small script
  block) that fetches/reads `project-data.json` and renders project cards
  into the DOM, mapping each field to the markup/classes Designer expects
  (`.project-card`, status badge element, priority element).
- After Designer's structural pass, re-verify the JS still targets the
  correct selectors/IDs and fix any wiring breaks introduced by markup
  changes.
- Create `.vscode/launch.json`:
  - Strict JSON, no comments.
  - `cwd`: `${workspaceFolder}/app`.
  - Launch a local static server rooted at `app/`.
  - Open `index.html` explicitly (not a bare directory listing).
  - Name the configuration exactly **"Run Project Pulse Dashboard"**.
  - Use deterministic port/URL values.
- Validate that opening the dashboard via the launch config shows the
  Project Pulse UI with data populated and no console errors.

## 5. Dependencies

1. **Data schema before consumption:** `app/project-data.json` schema
   (the `projects` array shape) must be defined before the JS in
   `app/index.html` writes any code that reads/maps fields from it.
   Coder owns both, so this is an internal Coder-side ordering, not a
   cross-role blocker.
2. **Base HTML skeleton before Designer's structural pass:** Designer's
   refinement of `app/index.html` (accessibility, class hooks) requires
   Coder's initial skeleton and rendering logic to exist first, so there
   is real markup/DOM structure to refine rather than designing in a
   vacuum.
3. **Class hooks before final CSS polish:** Designer's `app/styles.css`
   component styling (cards, badges, priority states) depends on the
   final agreed-upon class names in `app/index.html`. Designer should
   settle on `.dashboard` / `.project-card` (and any badge/priority class
   names) during the HTML structural pass before writing exhaustive CSS
   for those components, otherwise CSS selectors may not match.
4. **`.vscode/launch.json` depends on `app/` existing:** the launch
   configuration's `cwd` and served path assume `app/index.html` exists
   as the entry point, so `index.html` should exist (at least the Coder's
   base skeleton) before launch.json is authored, though the JSON content
   itself does not depend on the *content* of index.html — only its
   presence at `app/index.html`.
5. **No dependency between `project-data.json` and `styles.css`:** CSS
   styling does not need to read or depend on the data file's values
   (only on class names present in markup), so these two files have no
   data dependency between Designer and Coder.

## 6. Parallel work decisions

### Can run in parallel
- **Coder: draft `app/project-data.json` schema/data** — can be done at
  the same time as —
- **Designer: draft the design system tokens (palette, typography scale,
  spacing scale) as a plan/reference, before touching `styles.css` file
  content that depends on final class names.**
  - Justification: no file overlap (`project-data.json` vs. design-token
    planning notes) and no data dependency (data values don't affect
    color/typography choices).
- **Coder: author `.vscode/launch.json`** can run in parallel with
  **Designer: styling work in `app/styles.css`**, once Coder's base
  `app/index.html` skeleton exists.
  - Justification: no file overlap (`launch.json` vs `styles.css`), and
    `launch.json` only depends on the *existence* of `app/index.html`,
    not on its final styled state.

### Must run sequentially
- **Coder's initial `app/index.html` skeleton → Designer's structural
  refinement of `app/index.html`.**
  - Justification: same file, direct overlap. Editing this file in
    parallel risks conflicting edits and undoing each other's markup
    changes.
- **Designer's finalized class hooks in `app/index.html` → Designer's
  full `app/styles.css` component styling.**
  - Justification: no file overlap, but there is a data/contract
    dependency — CSS selectors must match the final class names, so
    styling should be finalized only after markup structure is locked.
- **Coder's `app/project-data.json` schema → Coder's JS rendering logic
  in `app/index.html`.**
  - Justification: same-owner sequential dependency; JS field mapping
    code cannot be written correctly until the schema/field names are
    fixed.

## 7. Phase-by-phase breakdown

- **Phase 1 (sequential, Coder only):**
  - Coder defines `app/project-data.json` schema and sample data.
  - Coder builds the base `app/index.html` skeleton (`.dashboard`
    container, card loop scaffold) and JS that reads
    `project-data.json` and renders cards using placeholder classes.
  - Files touched: `app/project-data.json`, `app/index.html`.

- **Phase 2 (sequential, Designer only, depends on Phase 1):**
  - Designer refines `app/index.html` structure: semantic
    elements/landmarks, accessibility attributes, finalizes class names
    (`.dashboard`, `.project-card`, status/priority classes).
  - Files touched: `app/index.html`.

- **Phase 3 (parallel — no file overlap once Phase 2 is done):**
  - **Designer:** writes full `app/styles.css` (palette, typography,
    spacing, breakpoints, card/badge/priority states, `.dashboard` and
    `.project-card` selectors, polish via `border-radius`/`box-shadow`).
  - **Coder:** creates `.vscode/launch.json` (strict JSON, no comments,
    `cwd: ${workspaceFolder}/app`, serves `app/`, opens `index.html`,
    named "Run Project Pulse Dashboard").
  - Justification for parallel execution: `styles.css` and
    `launch.json` are disjoint files with no shared data dependency;
    both only need Phase 2's finalized `index.html` structure/class
    names to exist, which is already satisfied.

- **Phase 4 (sequential, joint verification):**
  - Coder re-checks that JS selectors still match Designer's final
    markup/classes after Phase 2/3 changes; fixes any wiring breaks.
  - Coder and Designer jointly validate the rendered dashboard (see
    Validation expectations below).
  - Files touched: `app/index.html` (Coder, minor JS fixes only, not
    Designer's structural/CSS work).

## 8. Edge cases to handle

- **Empty or malformed `project-data.json`:** JS should handle a missing
  or empty `projects` array gracefully (e.g., show an empty-state
  message) rather than throwing unhandled errors.
- **Unknown `status` or `priority` values:** CSS/JS should have a
  fallback badge style for status/priority values not explicitly
  designed for, so new/unexpected values do not break layout.
- **Long text values:** project `name`, `owner`, or `recentActivity`
  strings that are unusually long should not break card layout —
  CSS should handle text wrapping/truncation.
- **Small viewport widths:** verify the responsive breakpoint(s) truly
  collapse the grid to a single column and remain readable.
- **`file://` vs local server access:** `fetch()` of a local JSON file
  may fail under `file://` protocol due to CORS restrictions in some
  browsers — this is exactly why `.vscode/launch.json` must serve the
  app via a local HTTP server rather than opening `index.html` directly
  from disk.
- **Launch configuration serving the wrong directory:** if `cwd`/served
  root is not `app/`, the learner will see a repository-root directory
  listing instead of the dashboard — must be explicitly verified.
- **Port conflicts:** if the configured port is already in use, the
  launch may fail; use a deterministic but reasonably safe port and
  document it.

## 9. Validation expectations

- Launch the dashboard via the VS Code **"Run Project Pulse Dashboard"**
  configuration in `.vscode/launch.json` and confirm:
  - It opens directly to the Project Pulse UI at `index.html`, not a
    server directory listing.
  - `.vscode/launch.json` parses as strict, comment-free JSON.
- Confirm `app/project-data.json` parses as valid JSON with a top-level
  `projects` array, and every project object includes `name`, `owner`,
  `status`, `recentActivity`, and `priority`.
- Confirm the rendered dashboard shows one `.project-card` per entry in
  `project-data.json`, with visible status badges and priority treatment
  matching each project's data.
- Confirm `.dashboard` and `.project-card` CSS selectors exist in
  `app/styles.css` and are visibly applied (rounded corners, shadow,
  spacing, responsive behavior at narrow widths).
- Open the browser dev console and confirm there are no JavaScript
  errors while the dashboard loads and renders data.
- Resize the browser/viewport to confirm the responsive breakpoint(s)
  reflow the card grid correctly (e.g., single column on mobile widths).
- Manually cross-check a couple of edge-case data entries (e.g., an
  unusual `status` value) to confirm graceful fallback styling rather
  than a broken layout.

## 10. Open questions

- Should `project-data.json` include the brief's mentioned "short
  contributor-friendly summary" as an extra field per project, given the
  brief's explicit five-field list (`name`, `owner`, `status`,
  `recentActivity`, `priority`) does not name a summary field? Recommend
  confirming with the Orchestrator before adding fields beyond the fixed
  schema, to avoid mismatched validation expectations.
- What exact port and command should `.vscode/launch.json` use to serve
  `app/` (e.g., a Node-based static server task, Python's
  `http.server`, or a VS Code extension-based live server)? This should
  be decided by the Coder based on what's available in the dev
  container/environment, and documented explicitly in the launch config
  and final handoff notes.
- Should the Designer's accessibility pass (Phase 2) include specific
  ARIA roles/landmarks, or is semantic HTML5 alone sufficient for this
  small dashboard? Recommend the Designer decide during Phase 2 and note
  the decision in their handoff report.

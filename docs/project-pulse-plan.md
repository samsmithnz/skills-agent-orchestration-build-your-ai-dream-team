# Project Pulse — Implementation Plan

## 1. Summary

Project Pulse is a small static dashboard that gives Mona's contributors an at-a-glance view of active projects — their name, owner, status, recent activity, and priority — rendered as polished cards. It will be built as three static files under `app/` (`index.html`, `styles.css`, `project-data.json`) plus a `.vscode/launch.json` so a learner can start it from VS Code's **Run and Debug** panel via **Run Project Pulse Dashboard**. The launch configuration serves the `app/` directory with `python3 -m http.server 5500` and opens `index.html` directly (never a directory listing) using a `serverReadyAction`. The HTML fetches `project-data.json` at runtime and renders cards into a `.dashboard` container with deterministic `.project-card` hooks so styling, testing, and validation stay predictable.

## 2. File assignments

All four files below have non-overlapping ownership so Coder and Designer can work in parallel where noted.

### `app/index.html` — Coder
- Complete, valid HTML5 document with `<html lang="en">`, `<meta charset>`, and `<meta name="viewport" content="width=device-width, initial-scale=1">`.
- Exact page title text **"Project Pulse"** (in `<title>` and in a visible `<h1>`).
- `<link rel="stylesheet" href="styles.css">` in the `<head>`.
- Semantic structure: `<header>` with title and short tagline, `<main>` containing a `<section class="dashboard">` wrapper, and inside it a container (e.g. `<ul class="project-list">` or `<div class="project-grid">`) that holds project cards.
- A small inline `<script>` (module or classic) that:
  - Fetches `./project-data.json`.
  - Parses the top-level `projects` array.
  - Renders each project into an element with class `project-card` containing name, owner, status, `recentActivity`, and priority.
  - Applies deterministic modifier classes (e.g. `status-active`, `priority-high`) derived from the data so CSS can style them.
  - Handles empty arrays and fetch/parse errors gracefully (see Coder responsibilities).
- Deterministic CSS hooks required: `.dashboard`, `.project-card`, plus predictable child hooks such as `.project-card__name`, `.project-card__owner`, `.project-card__status`, `.project-card__activity`, `.project-card__priority`.
- Must reference both `styles.css` and `project-data.json` explicitly (validator greps for these).

### `app/styles.css` — Designer
- Owns all visual styling. See Section 3 for full responsibilities.
- Must include, at minimum, selectors `.dashboard` and `.project-card`, and use `border-radius` and `box-shadow` (required by the automated validator).
- No JavaScript logic, no HTML, no data. Pure CSS.

### `app/project-data.json` — Coder
- Strict, parseable JSON (no comments, no trailing commas).
- Top-level shape:
  ```json
  { "projects": [ { ... }, { ... } ] }
  ```
- Each project object includes at least these fields (validator greps for the field names):
  - `name` (string)
  - `owner` (string)
  - `status` (string; recommended controlled vocabulary: `"Active"`, `"At Risk"`, `"Blocked"`, `"Complete"`, `"Planning"`)
  - `recentActivity` (string — short human-readable summary; may also include a machine date, see Open Questions)
  - `priority` (string; recommended controlled vocabulary: `"High"`, `"Medium"`, `"Low"`)
- Recommended additional fields for richer UI (not required by validator): `summary` (short contributor-friendly blurb), `progress` (0–100), `updated` (ISO date string).
- Ship 5–7 realistic sample projects to exercise layout, long names, and each status/priority variant.

### `.vscode/launch.json` — Coder
- **Strict JSON, no comments, no trailing commas** — the validator runs `python3 -m json.tool` against it.
- Standard shape: `{ "version": "0.2.0", "configurations": [ ... ] }`.
- One configuration with:
  - `"name": "Run Project Pulse Dashboard"` (exact string; validator checks it in the file, in the step doc, and in the handoff).
  - `"type": "node-terminal"` (recommended — lets us run a shell command without needing a debug adapter; alternative is `"type": "debugpy"` running the `http.server` module, but `node-terminal` is simplest and matches the "serve from app directory" requirement).
  - `"request": "launch"`.
  - `"command": "python3 -m http.server 5500"`.
  - `"cwd": "${workspaceFolder}/app"` — required so the server's document root is `app/` and `/` resolves to `index.html`, not a repo directory listing.
  - `"serverReadyAction"`:
    - `"pattern": "Serving HTTP on .* port ([0-9]+)"` (captures the port from the CPython `http.server` startup line).
    - `"uriFormat": "http://localhost:%s/index.html"` — required literal (validator greps for it) and guarantees the browser lands on the dashboard, not a directory index.
    - `"action": "openExternally"` (or `"debugWithChrome"` — see Open Questions; `openExternally` is safest for Codespaces port forwarding).
- Port `5500` is intentional and deterministic; if it conflicts, learners can change it, but the plan targets 5500.

## 3. Designer responsibilities

Scope: `app/styles.css` (owned), plus **advisory** input on `app/index.html` markup structure and class names before Coder finalizes it.

### Visual hierarchy
- Prominent page header with the "Project Pulse" title and a short tagline/subtitle.
- Cards are the dominant visual unit. Card title (project name) is the strongest element inside each card; owner and metadata are secondary; status and priority badges are visually distinct chips.
- Use a consistent type scale (e.g., 1rem base, ~1.75–2rem H1, ~1.15rem card title). Comfortable line-height (1.4–1.6) and generous padding inside cards.

### Status and priority visual language
- **Status badges** — colored chip with text label. Suggested palette (must meet WCAG AA on chosen background):
  - Active → green
  - At Risk → amber/orange
  - Blocked → red
  - Complete → neutral/blue
  - Planning → slate/gray
- **Priority** — either a colored left border on the card or a labeled pill:
  - High → strong accent (red/orange)
  - Medium → amber
  - Low → muted/neutral
- Do not rely on color alone — always pair color with a text label or icon (accessibility requirement).

### Card treatment
- `border-radius` (required by validator), soft `box-shadow` (required), subtle border or background contrast against page background, hover/focus elevation change.

### Responsive behavior
- Mobile-first single-column layout.
- Use CSS Grid with `grid-template-columns: repeat(auto-fill, minmax(280px, 1fr))` (or similar) on `.dashboard`'s card container so cards reflow from 1 → 2 → 3+ columns as viewport grows.
- Test at ~360px, ~768px, ~1200px widths.

### Accessibility
- Sufficient contrast (WCAG AA ≥ 4.5:1 for body text, ≥ 3:1 for large text and non-text UI).
- Visible `:focus-visible` outlines on any interactive element (links, buttons).
- Respect `prefers-reduced-motion` for any hover transitions.
- Guide Coder to use semantic elements: `<header>`, `<main>`, `<section>`, headings in order (`h1` → `h2` for cards), and `<ul>`/`<li>` for the card collection.

### Consistency
- Define a small set of CSS custom properties at `:root` for colors, spacing, radius, and shadows so status/priority variants stay coherent.
- Keep selectors flat and BEM-ish (`.project-card__status`, `.project-card--priority-high`) so Coder can emit matching classes deterministically.

## 4. Coder responsibilities

### `app/project-data.json` — data schema
- Top-level object with a single `projects` array (required).
- Each project object has (required for validator): `name`, `owner`, `status`, `recentActivity`, `priority`.
- Optional but recommended: `id` (stable slug), `summary`, `progress` (integer 0–100), `updated` (ISO 8601 date).
- Provide 5–7 sample entries covering every `status` and `priority` variant Designer plans to style, plus at least one long project name to stress layout.

### `app/index.html` — rendering approach
- Load data via `fetch('./project-data.json')` inside a small inline module script.
- Render into a container inside `.dashboard`. Each card is an element with class `project-card` plus deterministic modifier classes derived from data (lowercased, hyphenated), e.g. `project-card project-card--status-at-risk project-card--priority-high`.
- Include a small template function that returns an `HTMLElement` (or template string with proper escaping) — **HTML-escape all string fields** before insertion to prevent injection from malformed data.
- Provide stable child hooks (`project-card__name`, `__owner`, `__status`, `__activity`, `__priority`) for CSS and future tests.

### Error handling
- If `fetch` fails or response is not OK: render an accessible error region (`role="alert"`) inside `.dashboard` with a clear message ("Could not load project data.") and log the error to the console with context.
- If JSON parse fails: same error path.
- If `projects` is missing, not an array, or empty: render an empty state ("No projects to show yet.") inside `.dashboard` instead of leaving the page blank.
- Never leave the user staring at a blank page or an unhandled promise rejection.

### Deterministic IDs / classes for testability
- Required class hooks (validator- and test-friendly): `.dashboard`, `.project-card`.
- Recommended additional hooks: `.project-list` (container), `.project-card__name`, `.project-card__owner`, `.project-card__status`, `.project-card__activity`, `.project-card__priority`, `.dashboard__empty`, `.dashboard__error`.
- Card modifier classes derived from data must use a deterministic transform (lowercase + spaces→hyphens) so `"At Risk"` → `project-card--status-at-risk`.

### `.vscode/launch.json` — launch details
- `name`: `"Run Project Pulse Dashboard"` (exact).
- `type`: `"node-terminal"` (runs a shell command in a terminal; simplest match for the http.server requirement).
- `request`: `"launch"`.
- `command`: `"python3 -m http.server 5500"`.
- `cwd`: `"${workspaceFolder}/app"` — non-negotiable; guarantees `/` maps to `app/index.html`.
- `serverReadyAction`:
  - `pattern`: `"Serving HTTP on .* port ([0-9]+)"`.
  - `uriFormat`: `"http://localhost:%s/index.html"` (must be present verbatim).
  - `action`: `"openExternally"`.
- Validate with `python3 -m json.tool .vscode/launch.json` after writing.

## 5. Ordered implementation steps

1. **Confirm agreed contract.** Freeze: file list, JSON schema (`projects` array with `name`/`owner`/`status`/`recentActivity`/`priority`), class hook names (`.dashboard`, `.project-card`, `.project-card__*`, modifier convention), and status/priority controlled vocabularies.
2. **Coder — data.** Create `app/project-data.json` with 5–7 sample projects covering all status and priority variants and at least one long name.
3. **Coder — markup + rendering.** Create `app/index.html` with semantic structure, `.dashboard` container, fetch/render script, error and empty states, and deterministic class hooks.
4. **Designer — styles.** Create `app/styles.css` targeting the agreed hooks: base layout, `.dashboard` grid, `.project-card` visuals (`border-radius`, `box-shadow`), status/priority variants, responsive breakpoints, accessibility (focus, contrast, reduced motion).
5. **Coder — launch config.** Create `.vscode/launch.json` with the exact configuration in Section 4. Validate with `python3 -m json.tool`.
6. **Integration pass.** Coder + Designer verify rendered output against real data: spacing, wrapping of long names, badge legibility, keyboard focus. Designer polishes CSS; Coder tweaks class emission if needed.
7. **Manual launch validation.** Run **Run Project Pulse Dashboard** from VS Code's Run and Debug panel. Confirm the browser opens `http://localhost:5500/index.html` and shows the dashboard, not a directory listing. Stop the server.
8. **Handoff.** Hand results back to the Orchestrator for the final review/handoff step.

## 6. Dependencies between steps

- Step 1 (contract) blocks **all** subsequent work — schema and class hooks must be agreed first.
- Step 3 (markup) depends on Step 1 for hook names and on Step 2's schema (but not on Step 2's file existing — Coder can build markup against the schema in parallel with authoring the JSON).
- Step 4 (styles) depends only on Step 1's agreed class hooks. It does **not** need Steps 2 or 3 to finish to start drafting.
- Step 5 (`launch.json`) depends on `app/index.html` existing at the path the server will serve (Step 3 complete) so the manual launch test is meaningful. The JSON file itself can be authored earlier, but it should not be considered "done" until Step 3 lands.
- Step 6 (integration polish) requires Steps 3 and 4 complete.
- Step 7 (manual launch) requires Steps 3, 4, and 5 complete.
- Step 8 (handoff) requires Step 7 to pass.

## 7. Parallel work decisions

**Can run in parallel (after Step 1):**
- Coder authoring `app/project-data.json` (Step 2) and `app/index.html` (Step 3).
- Designer drafting `app/styles.css` (Step 4) against the agreed class hooks — no file overlap with Coder.

**Must run sequentially:**
- Step 1 (contract freeze) before anything else — otherwise Designer's selectors and Coder's classes will drift.
- Step 5 (`.vscode/launch.json` final validation) after Step 3 (`app/index.html` exists at `app/`).
- Step 6 (integration/polish) after Steps 3 and 4 are both in — final CSS tuning against real rendered markup catches issues (badge wrapping, long-name overflow) that drafts miss.
- Step 7 (manual launch) after 3, 4, and 5.

## 8. Validation expectations

Automated (mirrors `scripts/validate-exercise.sh` and the Step 3 workflow):
- `.vscode/launch.json` parses under `python3 -m json.tool`.
- `.vscode/launch.json` contains the exact string `Run Project Pulse Dashboard`.
- `.vscode/launch.json` contains `http://localhost:%s/index.html`.
- `app/index.html` contains `index.html` reference target, the string `Project Pulse`, and `project-card`.
- `app/styles.css` contains selectors `.dashboard` and `.project-card`, plus `border-radius` and `box-shadow`.
- `app/project-data.json` parses as JSON and includes a top-level `projects` key with objects containing `name`, `owner`, `status`, `recentActivity`, `priority`.

Manual:
- Launching **Run Project Pulse Dashboard** opens a browser to `http://localhost:5500/index.html` showing the dashboard — **not** a directory listing of `app/`.
- Cards render from `project-data.json` (change a name in JSON, reload → UI reflects it).
- Layout reflows from 1 → 2 → 3+ columns as the viewport widens.
- Browser DevTools console shows no errors on load.
- Accessibility spot checks: heading order (single `h1`, card titles `h2`), visible focus outlines, badges legible without relying on color alone, sufficient contrast on all text.

## 9. Edge cases

- **Empty `projects` array** — Render a friendly empty state inside `.dashboard`, not a blank page.
- **Missing or malformed `project-data.json`** — Fetch failure or JSON parse error must render an accessible error region (`role="alert"`) with a clear message and log details to the console.
- **Missing optional fields** on a project (e.g., no `summary` or `updated`) — Skip gracefully; do not render "undefined".
- **Unknown `status` or `priority` value** — Fall back to a neutral badge style rather than breaking layout.
- **Very long project names or activity strings** — CSS must wrap or truncate gracefully (`overflow-wrap: anywhere` or a clamp with title tooltip).
- **Many projects (20+)** — Grid should keep working; no fixed height on the dashboard container.
- **Small viewports (~320–360px)** — Single column, cards remain readable, badges wrap without breaking card edges.
- **Port 5500 already in use** — Learner may see a bind error; document that they can change the port in `launch.json` (both `command` and let `serverReadyAction` pattern pick it up).
- **Codespaces port forwarding** — `openExternally` is preferred over `debugWithChrome` because the forwarded URL differs from `localhost` inside Codespaces; the browser action should still land on the forwarded `/index.html`.
- **Special characters in data** — All rendered strings must be HTML-escaped by the render script.

## 10. Open questions

1. **`recentActivity` shape** — Single human-readable string (matches the brief and validator), or a structured object with `{ text, date }`? Recommendation: keep it a single string for the exercise; add optional `updated` ISO date alongside if a machine-sortable timestamp is desired.
2. **Launch action target** — `openExternally` vs `debugWithChrome` for `serverReadyAction.action`. Recommendation: `openExternally` for best Codespaces compatibility. Confirm with learner if they want DevTools attached.
3. **Launch type** — `node-terminal` (recommended, simplest) vs `debugpy` module launch vs a task-based approach. All can satisfy the requirements; `node-terminal` needs no extensions beyond built-ins.
4. **Controlled vocabularies** — Confirm the exact allowed values for `status` and `priority`. Plan assumes `Active | At Risk | Blocked | Complete | Planning` and `High | Medium | Low`.
5. **Number of sample projects** — Plan assumes 5–7. Confirm if a specific count is expected.
6. **Progress field** — Include a visual progress bar per card, or keep cards text-only? Plan treats `progress` as optional/nice-to-have.
7. **Filtering/sorting UI** — Out of scope for v1? Plan assumes yes (static render only).

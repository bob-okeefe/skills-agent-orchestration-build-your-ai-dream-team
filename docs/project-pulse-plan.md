# Project Pulse — Implementation Plan

> Planner output for Mona's "Project Pulse" dashboard. This plan is intended for
> the Orchestrator to break into phases and delegate to the Designer and Coder
> specialist agents. No code is produced here.

## 1. Summary

**Project Pulse** is a small static frontend dashboard that gives Mona's
contributors an at-a-glance view of active projects: who owns them, their
current status, recent activity, and priority/risk level. It is a polished,
card-based UI rendered from a local JSON data file and previewed via a VS Code
launch configuration.

**Intended outcome**

- A learner can press **Run Project Pulse Dashboard** in VS Code and immediately
  see the Project Pulse UI (not a directory listing).
- The dashboard visibly reads from `app/project-data.json` and renders one card
  per project with a status badge, priority treatment, owner, recent activity,
  and short summary.
- The layout looks like a real dashboard: header, grid of project cards,
  responsive behavior, accessible contrast, and deterministic CSS hooks.
- Designer and Coder work in **non-overlapping file scopes** so the Orchestrator
  can parallelize safely.

## 2. File assignments

File scopes are exclusive. The Designer never edits Coder files and vice versa.

| File | Owner | Purpose |
| --- | --- | --- |
| `app/index.html` | **Coder** | Semantic HTML skeleton with deterministic hooks (IDs/classes) that the Designer's CSS targets. Links `styles.css` and `app.js`. No inline styles. |
| `app/app.js` | **Coder** | Fetches `project-data.json`, renders project cards into the DOM using the agreed hooks, handles empty/error states. |
| `app/project-data.json` | **Coder** | Authoritative sample data. Top-level `projects` array conforming to the agreed schema. |
| `app/styles.css` | **Designer** | All visual styling: layout, typography, spacing, color system, cards, badges, priority treatment, responsive behavior, focus states. |
| `.vscode/launch.json` | **Coder** | Strict-JSON VS Code launch config named **Run Project Pulse Dashboard**, `cwd` = `${workspaceFolder}/app`, opens `index.html`. |
| `docs/project-pulse-plan.md` | **Planner** (this doc) | This plan. |

**Rule:** the Designer must not modify `index.html`, `app.js`, `project-data.json`,
or `.vscode/launch.json`. The Coder must not modify `styles.css`. If a hook or
class name needs to change, that change is coordinated by the Orchestrator as a
contract update, not a unilateral edit.

## 3. Designer responsibilities

The Designer owns `app/styles.css` only.

### Deliverables

1. A polished dashboard visual, not a bare HTML page.
2. A small, documented design system inside `styles.css` (CSS custom properties
   for color, spacing, radii, shadow, typography scale).
3. Dashboard shell (header + main grid) and card design.
4. Status badge system and priority treatment.
5. Responsive behavior from ~320px up to wide desktop.
6. Accessible focus, contrast, and motion behavior.

### Required CSS hooks (contract with Coder)

The Designer's stylesheet must style, at minimum, these deterministic hooks. The
Coder must emit exactly these hooks in `index.html` / `app.js`.

- `.dashboard` — outermost dashboard container.
- `.dashboard__header` — top header region (title, tagline).
- `.dashboard__grid` — responsive grid of cards.
- `.project-card` — individual project card.
- `.project-card__title` — project name.
- `.project-card__owner` — owner line.
- `.project-card__summary` — short contributor-friendly summary.
- `.project-card__activity` — recent activity line.
- `.status-badge` — base status badge.
- `.status-badge--on-track`, `.status-badge--at-risk`, `.status-badge--blocked`,
  `.status-badge--complete`, `.status-badge--unknown` — status variants.
- `.priority`, `.priority--low`, `.priority--medium`, `.priority--high`,
  `.priority--critical`, `.priority--unknown` — priority variants.
- `.empty-state` — shown when there are no projects.
- `.error-state` — shown when data fails to load.

### Style system expectations

- **Typography:** a single sans-serif system stack, clear scale for h1/h2/body/small.
- **Spacing:** 4px base scale exposed as `--space-1` … `--space-6`.
- **Color:** neutral background, elevated card surface, accessible text contrast
  (WCAG AA), plus semantic color tokens for status and priority.
- **Cards:** rounded corners, subtle shadow, hover elevation, readable padding.
- **Badges:** pill-shaped, high-contrast text on colored backgrounds.
- **Priority:** visible treatment (e.g., left accent border or colored label)
  that is distinguishable **without relying on color alone** (add a text label
  or icon character).
- **Responsive:** grid collapses from multi-column to single-column on narrow
  viewports; header wraps gracefully.
- **Accessibility:** visible `:focus-visible` outlines, contrast ≥ 4.5:1 for
  text, `prefers-reduced-motion` respected for any transitions.

## 4. Coder responsibilities

The Coder owns `app/index.html`, `app/app.js`, `app/project-data.json`, and
`.vscode/launch.json`.

### 4.1 `app/index.html`

- HTML5 doctype, `<html lang="en">`, meta charset, viewport meta.
- `<title>Project Pulse</title>`.
- Link `styles.css`; load `app.js` with `defer`.
- Semantic structure using the agreed hooks:
  - `<main class="dashboard">`
    - `<header class="dashboard__header">` with `<h1>Project Pulse</h1>` and a
      short tagline.
    - `<section class="dashboard__grid" id="projects" aria-live="polite"></section>`
    - `<template id="project-card-template">` containing the card markup with
      empty slots for title, owner, status badge, priority, activity, summary.
  - Fallback `.empty-state` and `.error-state` nodes (hidden by default) that
    `app.js` toggles.
- No inline styles. No hard-coded project data.

### 4.2 `app/app.js`

- On `DOMContentLoaded`, `fetch('./project-data.json')`.
- Validate the response: must be JSON with a top-level `projects` array.
- For each project, clone the `<template>` and populate the hooks.
- Map `status` → badge variant class using a known allow-list; fall back to
  `.status-badge--unknown` and display the raw value as a label.
- Map `priority` similarly with `.priority--unknown` fallback.
- Escape/assign text via `textContent` (never `innerHTML`) to avoid injection.
- Handle:
  - Empty `projects` array → show `.empty-state`.
  - Fetch failure or invalid JSON → show `.error-state` with a short message and
    log the error to `console.error`.
- No external dependencies. Vanilla JS only.

### 4.3 `app/project-data.json`

Schema (top-level `projects` array; each item):

| Field | Type | Notes |
| --- | --- | --- |
| `name` | string | Required. Project name shown as card title. |
| `owner` | string | Required. Person or team responsible. |
| `status` | string | Required. Expected values: `on-track`, `at-risk`, `blocked`, `complete`. Unknown values render with `unknown` variant. |
| `recentActivity` | string | Required. Short human-readable line, e.g., "Merged PR #42 yesterday". |
| `priority` | string | Required. Expected values: `low`, `medium`, `high`, `critical`. |
| `summary` | string | Optional. Short contributor-friendly description. Included to support the "short summary" brief item. |

Sample data must include **at least 4 projects** covering a mix of statuses and
priorities (including at least one `at-risk` or `blocked` and one `critical`)
plus at least one deliberately long title to exercise wrapping.

### 4.4 `.vscode/launch.json`

- Strict JSON. **No comments, no trailing commas.**
- `version`: `"0.2.0"`.
- One configuration named exactly **`Run Project Pulse Dashboard`**.
- `type`: `"chrome"` (falls back gracefully; `msedge` acceptable if repo pattern
  prefers it — see Open Questions).
- `request`: `"launch"`.
- `cwd`: `"${workspaceFolder}/app"`.
- `url`: deterministic — `"http://localhost:5173/index.html"` (port choice is
  deterministic; Orchestrator to confirm no conflict).
- If a preLaunch server is required, use a deterministic `preLaunchTask` that
  serves `${workspaceFolder}/app` on the same port. If the Codespace Simple
  Browser / Live Preview pattern is preferred by the environment, use `type:
  "node"` running a tiny static command, or defer to Open Questions below.

The **must-hold invariants** for this file regardless of server choice:

- `cwd` = `${workspaceFolder}/app`.
- The opened URL resolves to `index.html`, not a directory listing.
- Configuration name is exactly `Run Project Pulse Dashboard`.

## 5. Ordered implementation steps

| # | Step | Owner | Files touched |
| --- | --- | --- | --- |
| 1 | Confirm CSS-hook contract and JSON schema (this document). | Planner / Orchestrator | `docs/project-pulse-plan.md` |
| 2 | Author `project-data.json` sample data matching the schema. | Coder | `app/project-data.json` |
| 3 | Build `index.html` skeleton with agreed hooks and `<template>`. | Coder | `app/index.html` |
| 4 | Implement `app.js` rendering, empty state, error state. | Coder | `app/app.js` |
| 5 | In parallel with 3–4: implement `styles.css` design system, cards, badges, priorities, responsive layout, accessibility. | Designer | `app/styles.css` |
| 6 | Create `.vscode/launch.json` with deterministic name, `cwd`, URL. | Coder | `.vscode/launch.json` |
| 7 | Integration check: run the launch config, verify cards render, hooks match, no console errors. | Orchestrator | (read-only) |
| 8 | Address any hook mismatches via a coordinated contract update. | Orchestrator → Designer + Coder | as needed |

## 6. Dependencies between steps

- **Step 1 blocks everything else.** The CSS-hook contract and JSON schema must
  be frozen before Designer and Coder start, otherwise their outputs will not
  compose.
- **Step 2 must precede Step 4.** `app.js` renders against the JSON schema.
- **Step 3 must precede Step 5's final polish.** The Designer can start on the
  design system and tokens in parallel, but selector-level card/badge styling
  should target hooks that already exist in `index.html`. Because the hook
  contract is fixed in Step 1, Designer can safely write selectors against the
  contract even before Step 3 lands.
- **Step 6 depends on the app entry point being finalized** (`index.html` at
  `app/index.html`) and the chosen port.
- **Step 7 depends on Steps 2–6.**

## 7. Parallel vs sequential work

**Can run in parallel (non-overlapping file scopes):**

- Coder on `app/index.html` + `app/app.js` + `app/project-data.json`.
- Designer on `app/styles.css`.
- Both rely only on the shared contract from Step 1 (hook names + JSON schema).

**Must run sequentially:**

- Step 1 (contract) → everything.
- JSON schema (Step 2) → rendering logic (Step 4).
- Final app entry (`index.html`) → launch config (Step 6).
- Any change to the CSS-hook contract must be re-agreed by both agents through
  the Orchestrator before either edits their file — never a unilateral rename.

## 8. Edge cases to handle

- **Empty projects array** — `.empty-state` shown, header still renders.
- **Missing optional field** (`summary`) — element omitted or hidden, no layout
  break.
- **Missing required field** — render card with a visible placeholder (e.g.,
  "Unknown owner") rather than crashing the whole render loop.
- **Unknown `status` value** — `.status-badge--unknown` variant, raw label shown.
- **Unknown `priority` value** — `.priority--unknown` variant.
- **Very long title** — wraps within card; card does not overflow the grid.
- **Very long `recentActivity`** — truncates with ellipsis or wraps to max 2–3
  lines (Designer decides; must remain accessible to screen readers).
- **Small viewport (≤ 360px)** — single-column grid, header wraps, tap targets
  ≥ 44px.
- **Large viewport (≥ 1400px)** — grid caps at a reasonable max width so cards
  don't stretch unreadably.
- **`fetch` fails** (offline, wrong path, invalid JSON) — `.error-state` shown,
  `console.error` logged, no unhandled promise rejection.
- **Color-only signaling** — status and priority must also carry a text label.
- **`prefers-reduced-motion`** — disable non-essential transitions.
- **`prefers-color-scheme`** — optional; if Designer supports dark mode, it must
  keep AA contrast in both.

## 9. Validation expectations

The Orchestrator should verify all of the following before declaring the task
complete.

**Launch behavior**

- VS Code shows a launch config named **Run Project Pulse Dashboard**.
- Running it opens `index.html` from `app/`, not a directory listing.
- The `cwd` in `.vscode/launch.json` is exactly `${workspaceFolder}/app`.
- `.vscode/launch.json` parses as strict JSON (no comments, no trailing commas).

**Rendering**

- Dashboard header with the title "Project Pulse" is visible.
- One `.project-card` is rendered per entry in `project-data.json`.
- Each card shows: name, owner, status badge, priority treatment, recent
  activity, and (if present) summary.
- Status badges use the correct variant class per status.
- Priority treatment is visually distinct across levels and does not rely on
  color alone.

**Structure & hooks**

- `index.html` contains `.dashboard`, `.dashboard__header`, `.dashboard__grid`.
- Rendered DOM contains `.project-card`, `.status-badge`, `.priority` hooks.
- `styles.css` defines rules for every hook listed in Section 3.

**Responsiveness & accessibility**

- Layout is usable at 320px, 768px, and 1280px widths.
- Text contrast passes WCAG AA on all badges and body text.
- Keyboard focus is visible on any interactive element.
- No console errors or warnings when the page loads with valid data.
- With an empty `projects` array, `.empty-state` appears cleanly.
- With `project-data.json` renamed to force a fetch failure, `.error-state`
  appears cleanly.

## 10. Open questions

1. **Static server choice for `launch.json`.** VS Code's `chrome`/`msedge`
   launch types need a URL. Options:
   - Use a preLaunchTask that runs `python3 -m http.server 5173` from
     `${workspaceFolder}/app` (Python is present in the devcontainer — needs
     confirmation).
   - Use the **Live Preview** extension and a `simpleBrowser.show` command.
   - Use `npx serve` (requires Node + network).
   Orchestrator should confirm which is available in the Codespace before the
   Coder finalizes the launch config. The invariants (name, `cwd`, opens
   `index.html`) hold regardless.
2. **Port.** Proposed deterministic port is `5173`. Confirm no conflict with
   other Codespace services.
3. **Browser type.** `chrome` vs `msedge` for the launch `type` — pick whichever
   the exercise's existing docs or `.vscode` patterns prefer.
4. **Dark mode.** Should the Designer ship a `prefers-color-scheme: dark`
   variant, or is light-only sufficient for this exercise?
5. **Summary field.** The brief lists "a short contributor-friendly summary" but
   does not list `summary` in the required JSON fields. This plan treats it as
   an optional field on each project. Confirm.
6. **Number of sample projects.** This plan proposes ≥ 4. Confirm if a specific
   count is expected.
7. **Icons.** May the Designer use inline SVG for priority/status glyphs, or
   should it stay pure CSS + text to avoid new assets?

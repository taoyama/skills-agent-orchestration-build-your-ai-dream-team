# Project Pulse — Implementation Plan

## 1. Summary

Mona's team needs a lightweight **Project Pulse** dashboard: a small static frontend that loads project data from a local JSON file and renders a polished dashboard of project cards. Each card surfaces the project's name, owner, current status, recent activity, and priority/risk, using clear visual hierarchy, status badges, and priority indicators.

**Goals**

- Contributor-friendly, at-a-glance overview of active projects.
- Static app (no build step, no backend): `index.html` + `styles.css` + `project-data.json`.
- Runnable from VS Code via a **Run Project Pulse Dashboard** launch configuration that serves `app/` and opens `index.html` (never a directory listing).
- Deterministic CSS hooks (`.dashboard`, `.project-card`, badge/priority classes) so Designer and Coder work does not collide.
- Accessible, responsive, resilient to imperfect data.

## 2. Ordered Implementation Steps

1. **Lock the data contract** — finalize the JSON schema (`projects[]` with `name`, `owner`, `status`, `recentActivity`, `priority`) and the allowed enum values for `status` and `priority`. (Planner → Coder + Designer.)
2. **Author sample data** — Coder creates `app/project-data.json` with a representative `projects` array (mix of statuses, priorities, short/long strings, several rows).
3. **Scaffold markup + CSS hooks** — Coder creates `app/index.html` with the semantic skeleton and the agreed class hooks (`.dashboard`, `.dashboard__header`, `.project-list`, `.project-card`, `.status-badge`, `.status-badge--<status>`, `.priority`, `.priority--<level>`, `.empty-state`, etc.). Include the fetch/render script that reads `project-data.json` and renders cards.
4. **Style the dashboard** — Designer creates `app/styles.css` targeting the agreed hooks: layout grid, typography, spacing, badges, priority indicators, empty state, responsive breakpoints, focus states.
5. **Add launch configuration** — Coder creates `.vscode/launch.json` with the **Run Project Pulse Dashboard** configuration serving from `app/` and opening `index.html`.
6. **Integration pass** — Verify JSON parses, cards render, badges/priority styles apply, launch config opens the app (not a directory listing), responsive + a11y checks pass.
7. **Polish pass** — Designer refines visuals against real data; Coder addresses any minor markup gaps flagged by Designer.

## 3. File Assignments

### `app/project-data.json` — **Owner: Coder**
- Top-level object with a `projects` array.
- Each entry contains: `name` (string), `owner` (string), `status` (enum: e.g. `active`, `at-risk`, `blocked`, `on-hold`, `complete`), `recentActivity` (short string; may include a relative date), `priority` (enum: `low`, `medium`, `high`, `critical`).
- Include ≥ 6 sample projects spanning every status and priority to give Designer real content to style.
- Valid JSON (no comments, no trailing commas).

### `app/index.html` — **Owner: Coder**
- Semantic HTML5 document, `<html lang="en">`, meta viewport, title "Project Pulse".
- Links `styles.css`.
- Root layout: `<header class="dashboard__header">` (title, short tagline, optional summary counts) and `<main class="dashboard">` containing `<ul class="project-list">` (or `<section>` with cards) rendered from JSON.
- Card template classes (deterministic hooks for Designer):
  - `.project-card`
  - `.project-card__title`, `.project-card__owner`, `.project-card__activity`, `.project-card__summary`
  - `.status-badge` + modifier `.status-badge--{status}`
  - `.priority` + modifier `.priority--{low|medium|high|critical}`
- Inline (or `<script>`-tagged) vanilla JS that:
  - `fetch('./project-data.json')`, parses, iterates `projects`, builds cards via DOM APIs (no innerHTML of raw data — use `textContent` to avoid XSS).
  - Handles empty array → renders `.empty-state`.
  - Handles missing optional fields gracefully (fallback text like "Unassigned", "No recent activity").
  - Normalizes `status` / `priority` to a known set; unknown values get a `.status-badge--unknown` / `.priority--unknown` class.
  - Catches fetch/parse errors → renders `.error-state` message.
- Accessibility: landmarks (`header`, `main`), heading order, `aria-label` on the list, badges include visually-hidden text like "Status: at risk" for screen readers.

### `app/styles.css` — **Owner: Designer**
- Design tokens (CSS custom properties) for color, spacing, radius, shadow, typography scale, focus ring.
- Global reset/normalize basics, system font stack, base body styles.
- `.dashboard` layout: max-width container, padding, spacing rhythm.
- `.dashboard__header`: title hierarchy, subtitle, optional counts row.
- `.project-list`: responsive CSS grid (`repeat(auto-fill, minmax(280px, 1fr))`), gap.
- `.project-card`: card surface (background, border, radius, shadow), internal spacing, hover/focus elevation, clear title/owner/activity hierarchy.
- `.status-badge` + `--active|--at-risk|--blocked|--on-hold|--complete|--unknown`: pill shape, semantic colors with sufficient contrast (WCAG AA), never color-only (pair with label text).
- `.priority` + `--low|--medium|--high|--critical|--unknown`: distinctive indicator (dot/bar/chip) with label.
- `.empty-state`, `.error-state`: centered friendly messaging.
- Responsive breakpoints (small / medium / large).
- `:focus-visible` styles, `prefers-reduced-motion` respected, `prefers-color-scheme: dark` optional but recommended.

### `.vscode/launch.json` — **Owner: Coder**
- Configuration named exactly **Run Project Pulse Dashboard**.
- `cwd`: `${workspaceFolder}/app`.
- Opens `index.html` in the browser via a debug-capable configuration (e.g., `type: "chrome"` or `"msedge"` with `file` set to open `index.html` from the `app/` directory, OR a `serve`-style configuration that starts a static server rooted at `app/` and navigates to `/index.html`).
- Must NOT land the user on a directory listing — the URL/target must resolve to `index.html`.
- `version: "0.2.0"`, `configurations: [...]` shape.

## 4. Designer Responsibilities (Scoped)

- Own `app/styles.css` end-to-end.
- Define the visual language: tokens, typography scale, color system, spacing rhythm.
- Card layout and information hierarchy (title > owner > status/priority > activity).
- Status badge and priority indicator visual system (color + shape + label — never color alone).
- Responsive grid behavior and breakpoints.
- Accessibility polish: contrast, focus rings, reduced motion, optional dark mode.
- Empty-state and error-state visuals.
- Advise (not edit) on any HTML class-name additions required to achieve the design; those hooks are added by Coder.

## 5. Coder Responsibilities (Scoped)

- Own `app/index.html`, `app/project-data.json`, and `.vscode/launch.json`.
- Implement DOM rendering script (fetch → parse → render → error/empty handling).
- Guarantee the deterministic CSS hooks Designer depends on (`.dashboard`, `.project-card`, `.status-badge--*`, `.priority--*`, `.empty-state`, `.error-state`).
- Data normalization + safe text rendering (no XSS via `innerHTML`).
- Provide realistic sample data covering all status/priority variants.
- Configure the VS Code launch entry so it serves `app/` and opens `index.html`.
- Do not author visual styles; only structural or a11y-required attributes.

## 6. Dependencies Between Steps

- Step 1 (data contract) blocks Steps 2, 3, and 4 — Designer needs the enum values to plan badge/priority variants; Coder needs the schema for markup + JSON.
- Step 3 (markup with hooks) blocks Step 4 (styling) only for **final** styling; Designer can begin token/layout work in parallel using the agreed hook names.
- Step 2 (sample data) unblocks meaningful visual review in Step 4 and integration in Step 6.
- Step 5 (launch config) is independent of styling and can proceed once `app/index.html` exists (even as a stub).
- Step 6 (integration) requires Steps 2–5 complete.
- Step 7 (polish) requires Step 6.

## 7. Parallel vs Sequential Work

**Can run in parallel (non-overlapping file ownership):**
- After Step 1:
  - Coder: `app/project-data.json` (Step 2), `app/index.html` scaffold (Step 3), `.vscode/launch.json` (Step 5).
  - Designer: `app/styles.css` token layer + card/grid/badge/priority skeleton (Step 4) using the pre-agreed class hooks.
- These files have disjoint owners and paths → no merge conflicts expected.

**Must be sequential:**
- Step 1 → everything else (contract first).
- Step 3 scaffold must land before Designer's **final** pass in Step 4 (so real markup can be visually verified).
- Step 6 (integration) after Steps 2–5.
- Step 7 (polish) after Step 6.

## 8. Edge Cases to Handle

- **Empty `projects` array** → render `.empty-state` ("No projects to show yet").
- **Missing `project-data.json` / fetch error / invalid JSON** → render `.error-state` with a friendly message; log details to console.
- **Missing optional fields** (`owner`, `recentActivity`) → fallback text ("Unassigned", "No recent activity").
- **Unknown `status` or `priority` values** → apply `--unknown` modifier, still render label text.
- **Very long titles / owner names / activity strings** → CSS handles wrapping; consider `overflow-wrap: anywhere` and clamp lines for activity if needed.
- **Many projects (20+)** → grid should reflow; no pagination required, but layout must not break.
- **Very few projects (1)** → grid should not stretch a single card to full width awkwardly; use `minmax` + `auto-fill`.
- **Special characters in data** → rendered via `textContent`, not `innerHTML`.
- **Non-Latin text / long unicode** → font stack must render; test at least one entry.
- **User opens `app/` folder directly** → launch config must still target `index.html`, not a directory listing.
- **Reduced motion / dark mode users** → respect `prefers-reduced-motion` and (optionally) `prefers-color-scheme`.
- **Keyboard-only users** → visible focus on any interactive element; cards themselves need not be focusable unless they become links.

## 9. Validation Expectations

- **JSON validity:** `app/project-data.json` parses without error (e.g., `jq . app/project-data.json` succeeds).
- **Launch config:** Running **Run Project Pulse Dashboard** in VS Code opens the browser directly on the rendered dashboard (not a directory listing), served from `app/`.
- **Render check:** All sample projects appear as cards with name, owner, status badge, priority indicator, and recent activity.
- **CSS hooks present:** DOM contains `.dashboard`, `.project-card`, `.status-badge`, `.priority`, and the expected modifier classes; Designer's selectors match.
- **Responsive:** Layout looks correct at ~360px, ~768px, and ≥1200px widths; cards reflow, no horizontal scroll.
- **Accessibility basics:**
  - Landmarks (`header`, `main`) present, single `<h1>`, logical heading order.
  - Status/priority conveyed by text as well as color; contrast passes WCAG AA.
  - Visible `:focus-visible` outline on any interactive element.
  - Screen-reader labels for badges (visually hidden text or `aria-label`).
- **Resilience:** Manually blanking `projects` → empty state renders. Manually breaking JSON → error state renders, no white screen.
- **No console errors** on normal load.

## 10. Open Questions

- Exact allowed enum values for `status` — proposed: `active`, `at-risk`, `blocked`, `on-hold`, `complete`. Confirm with Mona.
- Exact allowed enum values for `priority` — proposed: `low`, `medium`, `high`, `critical`. Confirm.
- Should the header show aggregate counts (e.g., "3 active · 1 at risk")? Nice-to-have; assumed **yes, lightweight** unless declined.
- Dark mode: required or optional? Plan treats it as optional polish.
- `recentActivity` format: freeform string vs. structured (`{ text, timestamp }`)? Plan assumes freeform string for simplicity.
- Launch config style: static-server-based (e.g., `npx serve`) vs. `file://` open? Both meet the "open `index.html`, not a directory listing" requirement; Coder to pick the simplest that works in the devcontainer.
- Sorting/filtering of cards — out of scope for v1 unless Mona requests it.

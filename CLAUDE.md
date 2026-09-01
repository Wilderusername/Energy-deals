# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-file, static HTML prototype for **CanSpot** (internal `<title>`: "EnergyBoost — Prototyp"), a German-language mobile-first price-comparison app for energy drinks. There is no build step, no package manager, and no backend — `canspotprototype20260901.html` contains all CSS, inline SVG icons, and JavaScript in one file, driven by hard-coded demo data.

The filename is date-stamped (`canspotprototypeYYYYMMDD.html`); when starting a new dated iteration, copy the file forward rather than editing history in place unless the user says otherwise.

## Running / previewing

No build or install step — open the HTML file directly, or serve it locally. A dev server config already exists at `.claude/launch.json`:

```bash
python3 -m http.server 8123
```

(matches the `canspot-preview` launch config, serving the current directory on port 8123). There is no lint, test, or build command — this is plain HTML/CSS/JS with no tooling configured.

## Architecture

Everything lives in one file, in this order: `<style>` (CSS custom properties + component styles) → SVG `<symbol>` icon sprite → `<body>` markup (app shell, sheets/modals) → one `<script>` block with demo data and all logic.

**Theming**: CSS custom properties are defined on `:root` (light) and overridden under `:root[data-theme="dark"]`. Theme resolution (`light`/`dark`/`system`, from `localStorage["canspot-theme"]`) happens twice: once in a tiny inline `<script>` in `<head>` (sets `data-theme` before first paint to avoid a flash of the wrong theme) and again later via `applyTheme()` once the UI is interactive.

**Demo data model** (top of the main `<script>`):
- `products` — static catalog (name, brand, size, hotlinked image URL).
- `storeLogos` / `storeBranches` — fictional demo retailer branches (address, hours, geo) keyed by store name.
- `rawDeals` — hand-written base deals, extended programmatically (`newProductIds` loop) so every additional product gets deals at two stores.
- `deals` — final array: `rawDeals` joined with product info, plus a synthetically generated 90-day `history` (via `genHistory`) and a deterministic `checkedMinutesAgo` (via `hashStr`).
- `DEMO_TODAY` is a **fixed date** (currently `2026-09-01`), not `Date.now()` — all "days until expiry / days since start" logic (`daysUntil`) is relative to this constant so the demo data's validity windows stay consistent regardless of when the file is actually opened. Update this constant when the demo dates are rolled forward.

**State & rendering**: No framework — plain module-level `let` variables (`selectedBrand`, `favorites` (a `Set`), `radiusKm`, `viewMode`, `hideExpired`, etc.) hold UI state, mutated by event listeners attached directly via `addEventListener`/`querySelectorAll`. Each state change is followed by an explicit call to the relevant render function:
- `render()` — filters + sorts `deals`, rebuilds the `#deals` list (or shows an empty state), and also triggers `renderMap()`.
- `renderMap()` — draws the radial "map" view (store pins positioned by hashed angle + normalized distance, not real coordinates).
- `renderAlertsView()` — the "Alarme" tab (price-alert cards + computed favorite events).
- Views (list/map/alerts, and the profile/history/store-detail/location/filter/sort sheets) are all plain DOM containers shown/hidden — there is no router.

**Sheets/modals**: bottom-sheet overlays (`.overlay` + `.sheet`) are generic — `openSheet(id)` / `closeSheet(id)` toggle the `.open` class on any `#...Overlay` element. New sheets should follow the existing markup pattern (`.overlay > .sheet > .sheet-inner`, optional `.sheet-scroll` + `.sheet-footer` for tall/scrolling sheets).

**Persistence**: `localStorage` only, no server sync. Keys in use: `canspot-theme`, `canspot-alerts`, `canspot-name`, `canspot-email`, `canspot-avatar`, `canspot-notif-*`.

**Images**: product images and store logos are hotlinked directly from the official manufacturer/retailer websites (no local copies), with `onerror` handlers falling back to inline SVG data-URIs (`FALLBACK_IMG`, `STORE_FALLBACK_IMG`). This is called out explicitly in the in-app disclaimer text — preserve that disclaimer if the data source approach changes.

**Init sequence**: `renderSkeleton(4)` runs synchronously to show loading placeholders, then the real `render()` and related setup calls run inside a `setTimeout(..., 500)` at the very end of the script — this simulates a network-loading delay. Keep this in mind when scripting/automating against the page (content is not present until ~500ms after load).

## Workflow: progress log & commits

After every completed change, add a short entry to [PROJECT_STATE.md](PROJECT_STATE.md) (newest entry on top) covering: what was implemented, key decisions made, what's still open, and known bugs/next steps. `PROJECT_STATE.md` is the running progress log — `CLAUDE.md` stays reserved for durable project rules and technical notes and should not accumulate changelog entries.

After each meaningful milestone, create a git commit.

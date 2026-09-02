# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A static HTML prototype for **CanSpot** (internal `<title>`: "EnergyBoost — Prototyp"), a German-language mobile-first price-comparison app for energy drinks. There is no build step, no package manager, and no backend. `index.html` contains all CSS, inline SVG icons, and JavaScript in one file; it loads its offer data at runtime from [`deals.json`](#datenquelle-dealsjson) instead of embedding it inline.

## Running / previewing

No build or install step — open the HTML file directly, or serve it locally. A dev server config already exists at `.claude/launch.json`:

```bash
python3 -m http.server 8123
```

(matches the `canspot-preview` launch config, serving the current directory on port 8123). There is no lint, test, or build command — this is plain HTML/CSS/JS with no tooling configured.

## Architecture

Everything lives in one file, in this order: `<style>` (CSS custom properties + component styles) → SVG `<symbol>` icon sprite → `<body>` markup (app shell, sheets/modals) → one `<script>` block with demo data and all logic.

**Theming**: CSS custom properties are defined on `:root` (light) and overridden under `:root[data-theme="dark"]`. Theme resolution (`light`/`dark`/`system`, from `localStorage["canspot-theme"]`) happens twice: once in a tiny inline `<script>` in `<head>` (sets `data-theme` before first paint to avoid a flash of the wrong theme) and again later via `applyTheme()` once the UI is interactive.

**Data model** (top of the main `<script>`) — `products`, `storeLogos`, `storeBranches`, and `deals` are declared as empty `let` placeholders and populated at runtime by `loadDeals()` (see [Datenquelle (deals.json)](#datenquelle-dealsjson) below), not as inline literals:
- `products` — catalog (id, name, brand, sizeMl, hotlinked image URL), from `deals.json`'s `products[]`.
- `storeLogos` / `storeBranches` — retailer logo URL and branch info (address, hours, geo), derived from `deals.json`'s `stores{}` map (keyed by store name — this dataset models exactly one branch per chain in the demo area, not multiple branches per chain).
- `deals` — runtime array: `deals.json`'s `offers[]` joined with product info via `buildDeals()`, plus a `pfand`/`packaging` fallback (`PFAND_FALLBACK`/`PACKAGING_FALLBACK`, used only if an offer omits them), a `history` array (an offer's own `priceHistory` if present, else a synthetically generated 90-day series via `genHistory`), and a deterministic `checkedMinutesAgo` (via `hashStr`).
- `DEMO_TODAY` is a **fixed date** (currently `2026-09-01`), not `Date.now()` — all "days until expiry / days since start" logic (`daysUntil`) is relative to this constant so the demo data's validity windows stay consistent regardless of when the file is actually opened. Update this constant when the demo dates are rolled forward.

**State & rendering**: No framework — plain module-level `let` variables (`selectedBrand`, `favorites` (a `Set`), `radiusKm`, `viewMode`, `hideExpired`, etc.) hold UI state, mutated by event listeners attached directly via `addEventListener`/`querySelectorAll`. Each state change is followed by an explicit call to the relevant render function:
- `render()` — filters + sorts `deals`, rebuilds the `#deals` list (or shows an empty state), and also triggers `renderMap()`.
- `renderMap()` — draws the radial "map" view (store pins positioned by hashed angle + normalized distance, not real coordinates).
- `renderAlertsView()` — the "Alarme" tab (price-alert cards + computed favorite events).
- Views (list/map/alerts, and the profile/history/store-detail/location/filter/sort sheets) are all plain DOM containers shown/hidden — there is no router.

**Sheets/modals**: bottom-sheet overlays (`.overlay` + `.sheet`) are generic — `openSheet(id)` / `closeSheet(id)` toggle the `.open` class on any `#...Overlay` element. New sheets should follow the existing markup pattern (`.overlay > .sheet > .sheet-inner`, optional `.sheet-scroll` + `.sheet-footer` for tall/scrolling sheets).

**Persistence**: `localStorage` only, no server sync. Keys in use: `canspot-theme`, `canspot-alerts`, `canspot-name`, `canspot-email`, `canspot-avatar`, `canspot-notif-*`.

**Images**: product images and store logos are hotlinked directly from the official manufacturer/retailer websites (no local copies), with `onerror` handlers falling back to inline SVG data-URIs (`FALLBACK_IMG`, `STORE_FALLBACK_IMG`). This is called out explicitly in the in-app disclaimer text — preserve that disclaimer if the data source approach changes.

**Init sequence**: `loadDeals()` (at the very end of the script) calls `renderSkeleton(4)` synchronously to show loading placeholders, then `fetch("deals.json")`s the offer data. It waits for `Promise.all([fetch(...), delay(500)])` — the 500ms floor preserves the old simulated-loading feel so the skeleton doesn't just flash on a fast local response, while a genuinely slow/real request is still awaited fully rather than cut short. On success it populates `products`/`storeLogos`/`storeBranches`/`deals` and calls `finishInit()` (`render()` + the other setup calls that used to sit directly in the old `setTimeout`). On failure (network error, non-2xx, unexpected JSON shape) it calls `showLoadError()` instead, which renders a dedicated `.empty`-style error card with a "Erneut versuchen" button wired back to `loadDeals()`. Keep this in mind when scripting/automating against the page — content is not present until the fetch resolves (~500ms locally).

## Datenquelle (deals.json)

Products, stores/branches, and offers are **not** embedded in `index.html` — they're fetched at runtime from [`deals.json`](deals.json) (same-origin, relative path, so it works locally and under a GitHub Pages project subpath). This is the seam meant for swapping in a real offer feed later: replace the file's contents (or point `loadDeals()`'s `fetch()` at a different same-shape endpoint) without touching UI code, as long as the shape below is respected. `deals.json` currently contains only hand-authored demo data (its own `_meta.description` says so explicitly) — no live retailer data is fetched.

**Shape** (see `deals.json`'s own `_meta.fields` for the authoritative, versioned copy of this):
- `products[]` — `{ id, name, brand, sizeMl, img }`. `img` is a hotlinked product-image URL (see **Images** above).
- `stores{}` — keyed by retailer/chain name (the unified format's "Händler"). Each entry: `{ logo, branch: { address, geo, open, close, closedSun } }` — `branch` is the "Filiale" (address/hours/geo); `open`/`close` are hours in 24h float form (`7.5` = 7:30). One branch per chain in this dataset, not per-offer — a real multi-branch feed would need offers to carry their own `branchId`.
- `offers[]` — the unified **Angebotsformat**: `{ id, productId, store, regularPrice, offerPrice, pfand, packaging, distanceKm, validFrom, validUntil, link }`, plus an optional `priceHistory: number[]`. `productId` → `products[].id`; `store` → a `stores{}` key. `pfand` (EUR) and `packaging` ("Dose"/"Flasche") are the deposit/packaging fields; today every record sets `pfand: 0.25` and `packaging: "Dose"`, matching the previous hard-coded constants byte-for-byte (`PFAND_FALLBACK`/`PACKAGING_FALLBACK` in `index.html` only kick in if a record omits them). `validFrom`/`validUntil` (`YYYY-MM-DD`) are the "Angebotszeitraum"; `distanceKm` is the "Entfernung".
- **Öffnungsstatus is intentionally *not* stored** — `isStoreOpenNow()` computes it client-side from `stores{}.branch` (open/close/closedSun) against the current time on every render, so it can never go stale the way a snapshotted boolean would.
- **Preisverlauf**: `buildDeals()` uses an offer's own `priceHistory` verbatim if present; otherwise it falls back to the existing synthetic `genHistory()` curve (same as before this data source existed — none of the current demo offers set `priceHistory`, so today's chart output is unchanged, jitter included).
- `checkedMinutesAgo` ("Preis geprüft vor …") stays purely client-side, deterministic from the offer `id` via `hashStr()` — not sourced from JSON. A real feed would more naturally supply a `lastCheckedAt` timestamp; that's a deliberately deferred extension, not wired up.

**Loading states**: see **Init sequence** above for the loading/error mechanics. An empty (or all-filtered-out) `deals` array reuses the existing "Keine Angebote gefunden" empty state in `render()` — no separate "no data source" UI was needed.

## Progressive Web App (PWA)

The app is installable and works offline. This adds a few files alongside `index.html`, all wired with **relative** paths (no leading `/`) so it keeps working when served from a GitHub Pages project subpath (`https://<user>.github.io/<repo>/`), not just the domain root:

- `manifest.webmanifest` — name "CanSpot", `theme_color`/`background_color` matching the app's dark palette, `start_url`/`scope` set to `./`, and icon entries (see below).
- `icons/` — generated PNGs: `favicon-16.png`, `favicon-32.png`, `icon-192.png`, `icon-512.png` (purpose `any`), `icon-maskable-512.png` (purpose `maskable`, extra padding for Android's adaptive-icon safe zone), `apple-touch-icon-180.png`. All share the same mark — the can/location-pin glyph isolated from the logo SVG's standalone icon paths (i.e. the three `<path>` elements outside the lettering `<g>` in the header logo), white/`--brand-primary-blue` on a flat `--brand-primary-dark` background, rounded-square corners on every size except `icon-maskable-512` (full-bleed square; the OS applies its own mask) — matches the app's `theme-color`. Regenerated via a canvas-drawing script (`new Image()` from a data-URI SVG of just those 3 paths, drawn onto a `<canvas>` per size/corner-radius/mark-scale, exported with `toDataURL`) — there's no source vector file or generator script checked in, so redo this the same way if the mark changes again.
- `service-worker.js` — cache-first for same-origin requests, precaches the app shell (`index.html`, the manifest, `deals.json`, and the icons) on install, and falls back to the cached `index.html` for navigation requests when offline. It does **not** cache the hotlinked product/store images (external origins) — those still depend on network access, consistent with the existing hotlinking approach. Because `deals.json` is precached and the fetch handler is cache-first, an already-installed PWA keeps serving its last-cached offer data offline; a returning-online visitor also keeps seeing that cached `deals.json` until `CACHE_NAME` is bumped (see the bump note below) — this is deliberate, not a bug, but worth knowing when `deals.json` changes.
- In `index.html`'s `<head>`: `<link rel="manifest">`, favicon `<link>`s, `apple-touch-icon`, and the `apple-mobile-web-app-*` / `mobile-web-app-capable` meta tags for iOS/Android homescreen install. The service worker is registered at the very end of the main `<script>` block (after `loadDeals()` is kicked off), gated on `"serviceWorker" in navigator`, so it doesn't affect the existing init sequence or any UI behavior.
- **Mandatory cache-bump rule (no reminder needed):** `APP_SHELL` in `service-worker.js` lists every file the service worker precaches and serves cache-first (`index.html`, `manifest.webmanifest`, `deals.json`, all `icons/*.png`). Whenever a change touches the *content* of one of these files — including any edit inside `index.html`'s inline `<style>`, `<script>`, or markup, since it's the single-file app shell — bump `CACHE_NAME` in `service-worker.js` (currently `canspot-cache-v23`; next bump would be `v24`) as part of that same change, automatically, without being asked. The `activate` handler deletes old-named caches on its own once the version string changes. Without the bump, a returning visitor or already-installed PWA keeps being served the old cached files indefinitely — a `git push` updates GitHub Pages, but the change stays invisible on the live/test site until the cache is forced to invalidate. Only skip the bump for changes that touch nothing in `APP_SHELL` (e.g. only the standalone SEO landing pages, or `PROJECT_STATE.md`).

## Workflow: progress log & commits

Before treating any change as finished: check whether it touched a file listed in `service-worker.js`'s `APP_SHELL` (see **Progressive Web App (PWA)** above — in practice this means almost any `index.html` edit). If so, bump `CACHE_NAME` in the same change, so that a normal `git commit` + `git push` reliably updates the live GitHub Pages/test site instead of leaving visitors on a stale cached version. This applies automatically, every time, without the user needing to ask for it.

After every completed change, add a short entry to [PROJECT_STATE.md](PROJECT_STATE.md) (newest entry on top) covering: what was implemented, key decisions made, what's still open, and known bugs/next steps. `PROJECT_STATE.md` is the running progress log — `CLAUDE.md` stays reserved for durable project rules and technical notes and should not accumulate changelog entries.

After each meaningful milestone, create a git commit.

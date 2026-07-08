---
trigger: always_on
description: > **Read this entire file before touching any code.**
---

# Veriqo — Agent Instructions

> **Read this entire file before touching any code.**
> This is the only authoritative instruction file. CURRENT_STATE.md has been deleted — do not recreate it.

---

## ⚠ Critical rules

1. **Always `git pull origin main` before making any changes.** Multiple Claude Code sessions run against this repo (cloud + MacBook Air). Pushing stale code overwrites live fixes and breaks production.

2. **Source of truth is this GitHub repo** (`mogorman-a11y/mise`, `main` branch). Do NOT work from any local `files/` directory or use `vercel deploy --prod`. The old deploy chain is retired.

3. **Deploy chain:** push to `main` → Vercel auto-deploys → live at `getveriqo.co.uk`. No manual step.

4. **Bump `haccp.js` version on every change.** Update the `?v=N` query string in `app.html` each time `js/modules/haccp.js` is modified. Current version: **v51**. Do NOT bump `sw.js` cache name — the SW uses network-first so query string bumps are sufficient. **Every commit that touches haccp.js must bump the version — including bug fixes. Skipping this causes browsers to serve stale cached code.**

5. **Update this file** when versions change or architecture changes.

---

## Live versions

| File | Version | Where set |
|---|---|---|
| `js/modules/haccp.js` | `?v=51` | `app.html` script tag |
| `js/modules/menus.js` | `?v=21` | `app.html` |
| `js/modules/prep.js` | `?v=9` | `app.html` |
| `js/modules/dashboard.js` | `?v=5` | `app.html` |
| `sync.js` | — | no version param needed (SW network-first) |
| Service worker cache | `veriqo-v112` | `sw.js` line 8 |

---

## Architecture

- **Repo:** `mogorman-a11y/mise` on GitHub, `main` branch
- **Live site:** `getveriqo.co.uk/app` → served by Vercel, auto-deploy on push to `main`
- **Vercel project: `files`** (project ID `prj_lMBGlA1dkPtLSm3bUn9KZtAKpuWG`) — this is the only live Vercel project for Veriqo. The old `mise` and `mise-deploy` projects have been deleted. Do not recreate them.
- **No staging** — changes go live immediately on push.
- **Domain:** apex `getveriqo.co.uk` is canonical. `www.getveriqo.co.uk` redirects to the apex (set in Vercel Project → Settings → Domains, 2026-07-03). Every canonical tag, `og:url`, and JSON-LD `url`/`@id` across the site must use the apex, no `www` — do not reintroduce hardcoded `www.getveriqo.co.uk` links.
- **Supabase:** `https://yixrwyfodipfcbhjcszp.supabase.co`

### App structure

One unified PWA (`app.html`) with five modules: Dashboard, HACCP, Menus, Costing, Settings.

| Module | JS file | localStorage prefix |
|--------|---------|---------------------|
| HACCP  | `js/modules/haccp.js` | `haccp_` |
| Menus  | `js/modules/menus.js` | `mise_` |
| Prep Lists | `js/modules/prep.js` | — (Supabase only) |
| Costing | `js/modules/costing.js` | `yield_` |
| Dashboard | `js/modules/dashboard.js` | — |
| Sync   | `sync.js` | — |

### Key globals

- `records` — HACCP today's records array (module-level var in haccp.js, also `window.records`)
- `settings` — HACCP settings (module-level var in haccp.js)
- `mSettings` — Menus settings (`window.mSettings`, set in menus.js)
- `window.Mise.profile` — user profile (chef_name, business_name, etc.)
- `window.Mise.sync` — sync functions (saveDay, loadAll, etc.)

---


## Prep Lists module — key facts

**File:** `js/modules/prep.js` (v9)
**Tab:** `tab-prep` inside `#module-menus`
**Storage:** Supabase `prep_lists` table only — no localStorage

### Supabase table: `prep_lists`
- `user_id UUID NOT NULL` — **no default, must be supplied explicitly** in every INSERT (get from `supabaseClient.auth.getSession()`)
- `venue_id` — auto-populated via `auth_venue_id()` trigger
- `name`, `date`, `menu_id` — string fields
- `items JSONB` — array of item objects (see below)

### Item object shape
```javascript
{
  id: 'pi_...',           // unique, generated client-side
  dish_id: String,
  dish_name: String,
  dish_category: String,  // e.g. 'Starter', 'Main' — used for course sorting
  description: String,
  section: 'prep_ahead' | 'finishing',
  completed: Boolean,
  completed_at: ISO string | null,
  completed_by: UUID | null
}
```

### Course sort order
`_COURSE_ORDER = ['Canapé','Starter','Fish course','Main','Side','Sauce','Pre-dessert','Dessert','Cheese','Petit four','Bread','Other']`

`_courseIndex(item)` resolves `dish_category` from the item itself, then falls back to `mSettings.savedDishes` for older items that predate the `dish_category` field.

### Key functions
- `renderPrepIndex()` — entry point; called by `showTab('prep')`
- `openPrepListView(id)` / `closePrepListView()` — switches index ↔ list view; shows/hides `menus-back-btn` for mobile header
- `_renderPrepListView(id)` — renders full list with ← back link (embedded in content, works on desktop sidebar layout too)
- `_renderPrepSection(items, label, listId)` — renders one section with dish sub-headings grouped by course
- `tickPrepItem(listId, itemId)` — optimistic tick/untick, syncs to Supabase
- `editPrepItem / savePrepItemEdit / cancelPrepItemEdit` — inline edit a step
- `deletePrepItem(listId, itemId)` — removes step, syncs
- `deletePrepList(id)` — deletes whole list from Supabase + cache
- `resetPrepListTicks(id)` — unchecks all items (for reuse on next service)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mogorman-a11y/mise](https://github.com/mogorman-a11y/mise) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->

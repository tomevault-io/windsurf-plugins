---
trigger: always_on
description: - This is a **standalone React library** — it must work independently of DentalQuoteCreator
---

# React Advanced Odontogram

## Critical Defaults
- This is a **standalone React library** — it must work independently of DentalQuoteCreator
- Never introduce DentalQuoteCreator-specific dependencies or imports
- Preserve the public API (`OdontogramShell` props interface)
- All changes must maintain backward compatibility with existing integrations

## Architecture
- **React 18.3** + **TypeScript 5.5** — single-page interactive dental charting component
- **Vite 7.3** — dev server and library build
- **Tailwind CSS** — utility-first styling with custom theme via CSS variables (`--odon-*`)
- SVG-based tooth rendering with per-surface interaction

## Key Files
- `src/App.tsx` — main `OdontogramShell` component (exported as default)
- `src/odontogram.ts` — core odontogram state logic and types
- `src/plugin.ts` — plugin system for extending functionality
- `src/status_extras.ts` — additional dental status types (bridges, implants, etc.)
- `src/theme.ts` — theme configuration and CSS variable mapping
- `src/i18n/locales/<code>.ts` — the UI strings, one file per language (12). Only `en.ts` is bundled statically; `src/i18n/loader.ts` fetches every other language on demand (`loadLanguage`), and `src/i18n/useI18n.ts` holds `t()` (synchronous) and `setI18nLanguage()` (loads first, switches second). `src/i18n/translations.ts` imports ALL locales and is for tests only — never import its value from app code (`i18n-lazy-load.test.ts` guards this)
- `src/utils/numbering.ts` — tooth numbering systems (FDI, Universal, Palmer)
- `src/assets/teeth-svgs/` — SVG templates for individual teeth (11-48)
- `src/assets/icon-svgs/` — UI icons

## Tooth State Model
Each tooth has:
- `toothSelection`: tooth-base | missing | implant | pontic
- `toothSubstrate`: natural | radix | broken | crownprep
- `restorationType`: none | crown | inlay | onlay | veneer | bridge (implant-gated crowns/bridges compose with an implant connector layer). Restoration options are gated by tooth kind via `restorationOptions()` (`src/registry/restorations.ts`): an implant tooth offers only crown/bridge, plus the five `prosthesis` attachment entries; a missing/gap tooth (`toothSelection === "none"`) offers only a bridge pontic, plus the two removable-denture `prosthesis` entries; a `radix` substrate hides the restoration control entirely (`restorationRowHidden()` in `src/odontogram.ts`) — no restoration can be authored on a root remnant. A `bridge` restoration renders both the crown cap AND the saddle-connector layer via `composeRestorationLayers()`; the multi-tooth bridge-span overlay connector is arch-aware (mirrored saddle-Y fraction for the lower arch, `SADDLE_Y_FRACTION_LOWER` in `src/bridgeOverlay.ts`), and applying a bridge via a Statuses preset explicitly re-triggers the overlay recompute
- `restorationMaterial`: none | emax | gold | gradia | zircon | metal | metal-ceramic | telescope | temporary
- `prosthesis`: none | healing-abutment | locator | locator-denture | bar | bar-denture | removable-partial | removable-full — orthogonal implant-attachment / removable-denture axis, surfaced as "Kivehető:" entries in the combined restoration dropdown (gated per tooth kind — see `restorationType` above)
- `crownLeakage`: boolean — marginal-leakage finding, shown only when `restorationType` is crown or bridge. Both the tooltip (`getStateSummary`) and whole-mouth summary (`getOdontogramSummary`) gate this line on the exact same predicate as the `#crownLeakageRow` control (`!restorationRowHidden(state)` AND crown/bridge), so a tooth with a stale crown/bridge payload but a hidden restoration row (radix/milktooth/extraction/under-gum) never shows a leakage line the control itself doesn't display
- `endo`: none | endo-medical-filling | endo-filling | endo-filling-incomplete | endo-glass-pin | endo-metal-pin — mutually exclusive with `pulpDx`; both are surfaced through one merged "Pulp / Endo status" selector (vital vs. treated groups), and setting `endo` to a treated value normalizes `pulpDx` to `normal` (suppressing the diseased-pulp glyph)
- `pulpDx`: normal | reversible-pulpitis | irreversible-pulpitis | necrosis — AAE pulp diagnosis (replaced the retired `pulpInflam` boolean); mutually exclusive with `endo` (see above); `reversible-pulpitis` renders a reduced pulp glyph
- `pulpLatin`: none | pulpa-sana | hyperaemia-pulpae | pulpitis-acuta-serosa | pulpitis-acuta-purulenta | pulpitis-chronica-clausa | pulpitis-chronica-ulcerosa | pulpitis-chronica-hyperplastica | necrosis-pulpae | gangraena-pulpae — practical-Latin pulp subtypes, surfaced by the pulp picker only when the `pulpDetailLevel` setting (`simple` | `aae` | `latin`, default `aae`) is `latin`. `setPulpDetailLevel()` calls `notifyStateChange()`, so switching the level live-refreshes the whole-mouth summary and every tooth's tooltip immediately instead of staying stale until the next tooth edit

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ZoliQua/React-Advanced-Odontogram](https://github.com/ZoliQua/React-Advanced-Odontogram) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->

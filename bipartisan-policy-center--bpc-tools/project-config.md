---
trigger: always_on
description: This file documents the interactive tools, calculators, and page components built for
---

# BPC Interactive Tools — Claude Code Context

This file documents the interactive tools, calculators, and page components built for
bipartisanpolicy.org in Claude chat sessions that are being migrated into this repository.
Load this file in Claude Code for full project context before editing any file.

---

## Repository structure (proposed)

```
bpc-tools/
├── CLAUDE.md                        ← this file
├── calculators/
│   └── auto-loan-interest/
│       └── auto_interest_calc_embed.html
├── maps/
│   └── iso-rto/
│       └── isorto_map.html
├── tables/
│   └── state-pfml/
│       └── pfl_table.html
├── modals/
│   └── housing-bills-comparison/
│       └── housing_modal.html
└── internal-tools/
    ├── email-qa/
    │   └── email_qa_checker.html
    └── ga4-slug-explorer/
        └── bpc_slug_traffic.html
```

---

## Tool inventory

### 1. Auto Loan Interest Deduction Calculator
**File:** `calculators/auto-loan-interest/auto_interest_calc_embed.html`
**Status:** Live on bipartisanpolicy.org (EPP section)
**Purpose:** Interactive calculator for the 2026 auto loan interest deduction. Users input
income, loan amount, and interest rate; the tool computes estimated deduction and tax savings
with eligibility phaseout logic.

**Key implementation notes:**
- Scoped under `.bpc-auto-calc` wrapper to prevent CSS bleed into WP theme
- All element IDs prefixed with `bpc-` to avoid DOM collisions
- Uses document-level event delegation (not element queries) for WordPress compatibility
- `PHASEOUT_RATE` is `0.20` (updated from original `0.10` per EPP research team, April 2026)
- Versioned with changelog comments in the script block

**WordPress Custom HTML block constraints** (applies to ALL tools in this repo):
1. No `&&` — WP entity-encodes it to `&#038;&#038;`, breaking execution → use nested `if` statements
2. No raw non-ASCII characters (em dash, ▾, ▸, etc.) — use `\uXXXX` Unicode escapes
3. No apostrophes inside single-quoted JS strings → use double quotes or rephrase
- `||`, comparison operators (`<`, `>`, `<=`), and all standard ASCII survive intact

---

### 2. U.S. ISO/RTO Interactive Map
**File:** `maps/iso-rto/isorto_map.html`
**Status:** Built; embed status TBD
**Purpose:** Leaflet.js map of U.S. ISO/RTO electricity market regions with clickable regions,
detail panel (org name, states served, market type, emissions info, peak MW, population), and
legend with click-to-highlight.

**Key implementation notes:**
- Uses real ISO/RTO boundary GeoJSON (FERC/EIA source), simplified from ~31MB to ~550KB
  using Shapely — do not replace with the full-size file
- GeoJSON loaded at runtime from a relative path or CDN; confirm path is correct on deploy
- Seven regions covered: ISO-NE, PJM, MISO, SPP, NYISO, CAISO, ERCOT
- Companion Datawrapper-ready county-level CSV exists separately (not in this repo)
- BPC brand colors used for region fills; DM Sans for map UI font
- `Boundaries as of 2022` noted in panel footer

---

### 3. State PFML Comparison Table
**File:** `tables/state-pfml/pfl_table.html`
**Status:** Live on bipartisanpolicy.org (PFML explainer page)
**Purpose:** Tabbed HTML table comparing mandatory vs. voluntary state paid family and medical
leave programs across all 50 states. Tabs: All / Mandatory / Voluntary.

**Key implementation notes:**
- Data source: `2026-4_State_PFL_Comparisons_Table.xlsx` (BPC SharePoint)
- All `th` background/color rules require `!important` to override BPC WordPress theme
- Virginia appears in BOTH tabs:
  - Mandatory: added when Gov. Spanberger signed (late April 2026); job protection
    caveat is "Yes, if employed by current employer for 120+ days"
  - Voluntary: "Superseded" badge present — Virginia's 2022 voluntary law predates
    the 2026 mandatory program
- Footnote explains AWW, SAWW, Pending, Superseded abbreviations
- FAQ schema block (`FAQPage` JSON-LD) is a separate embed on the same page — not in this file
- `bpcPflTab()` JS function handles tab switching; uses `aria-selected` for accessibility
- Trailing `&nbsp;`/`<br>` WP artifacts addressed with a JS cleanup block at bottom

**Tab counts as of last update (April 23, 2026):**
- Mandatory: 14 states + D.C.
- Voluntary: 10 states

---

### 4. Housing Bills Comparison Modal
**File:** `modals/housing-bills-comparison/housing_modal.html`
**Status:** Live on bipartisanpolicy.org (housing bills page)
**Purpose:** Embeds a Datawrapper comparison chart (ID: cvs16, height 14,668px) in a
modal to avoid forcing excessive page scroll. Two side-by-side buttons trigger the modal
and link to a companion PDF download.

**Key implementation notes:**
- Modal has no title header — uses a floating `position:absolute` close button in top-right
  corner of the modal card
- Buttons: "View the Comparison Table" + "Download the Table"
  - Download links to: `https://bipartisanpolicy.org/wp-content/uploads/2026/04/Table-2.0-Policies-in-ROAD-and-21st-Century.pdf`
- Button styling: `padding: 14px 28px`, `font-size: 18px`, flex row centered
- Datawrapper iframe: `cvs16`, height hardcoded to `14668px` inside a fixed-height scrollable wrapper
- All WP Custom HTML constraints apply (no `&&`, no non-ASCII, no apostrophes in single-quoted strings)

---

### 5. Newsletter Email QA Checker *(internal tool)*

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Bipartisan-Policy-Center/bpc-tools](https://github.com/Bipartisan-Policy-Center/bpc-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->

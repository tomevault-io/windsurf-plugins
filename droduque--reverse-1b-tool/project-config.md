---
trigger: always_on
description: SVN Rock Advisors is a Canadian real estate development consulting firm based in Burlington, Ontario. They produce **feasibility studies** (called "1A") for apartment developers. The 1A includes a one-page Excel proforma with unit mix, rents, operating expenses, NOI, and valuation at three cap rates.
---

# SVN Rock — Reverse 1B Automation

## Project Context

SVN Rock Advisors is a Canadian real estate development consulting firm based in Burlington, Ontario. They produce **feasibility studies** (called "1A") for apartment developers. The 1A includes a one-page Excel proforma with unit mix, rents, operating expenses, NOI, and valuation at three cap rates.

After delivering the 1A, they present a detailed **Reverse 1B**, a 15-sheet Excel financial model that reverse-engineers the full development cost structure from that single proforma page. This currently takes their fractional CFO (Noor) 2-3 hours per project. The goal is to automate it so junior staff can generate it in minutes, then Noor reviews.

### The Business Flow
1. Client pays $20-25K for a feasibility study
2. The feasibility study contains a **1A proforma** (one-page Excel with rents, expenses, NOI, valuation)
3. SVN takes that 1A and builds a **Reverse 1B**, a 15-sheet financial model working backwards from the building's value
4. The Reverse 1B is presented to the client for FREE as a "wow" moment to demonstrate expertise
5. This leads to paid services: full 1B financial modeling ($5K), mortgage brokerage, lease-up, etc.

---

## Project Tracking

At the END of every session:
- Update guides/PROJECT-SUMMARY.md with what was built
- Mark completed items as done
- Update "Last Updated" date

---

## Strategic Reference (read before any Derek interaction)

- [notes/derek-9-steps-playbook.md](notes/derek-9-steps-playbook.md) — Distilled from the 10 ADFSE Deep Dive videos. Per-step Derek vocabulary, dollar hooks, metaphors. Drop a Derek-phrase per conversation.
- [notes/svn-rock-consulting-launch.md](notes/svn-rock-consulting-launch.md) — Patron-platform engagement model, the "final hook." Phase 1 (Movie Mode, awaiting Derek) → Phase 2 (SVN OS modules) → Phase 3 (equity). **Don't act on Phase 2 until Derek surfaces it.**
- Raw transcripts: [context/reference/adfse-deep-dive/](context/reference/adfse-deep-dive/)

---

## Key Files

```
reference/
├── 1A_Birchmount_2240.xlsx           # Project A — 1A proforma (170 units, 3 unit types)
│                                      # This is the SOURCE data that was used to build the template
├── 1A_490_St_Clair.xls               # Project B — 1A proforma (372 units, 9 unit types incl. affordable)
│                                      # TEST CASE — .xls format, needs conversion
├── REVERSE_1B_Template.xlsx          # The finished Reverse 1B for Birchmount (15 sheets, full formulas)
│                                      # Sheet 1 contains the Birchmount 1A data
├── Reverse 1B - Example & Inputs.xlsx # Spec sheet listing all ~100 input parameters with defaults
└── SAMPLE - 1B Model.xlsx            # The FULL 1B model (8 sheets) — reference only

context/reference/
└── 1B_User_Manual.pdf                # Official user manual with color conventions and sheet descriptions

guides/
├── DATA_MAP.md                       # Cell-by-cell mapping: 1A proforma → Reverse 1B template
├── INVESTIGATION_REPORT.md           # Sheet-by-sheet analysis of the template
└── PROJECT-SUMMARY.md                # Build progress tracking
```

---

## Critical Rules — Read These Before Writing Any Code

### Rule 1: NEVER Overwrite Formula Cells
The Reverse 1B template uses a color convention:
- **BLUE cells** = user inputs/assumptions, SAFE to write to
- **BLACK cells** = formulas, **DO NOT TOUCH**
- **GREEN cells** = formula but overridable, write with caution

Before writing to ANY cell, check if it contains a formula. If it does, skip it unless you're absolutely certain it's meant to be overwritten.

### Rule 2: Sheet 1 Is The Entry Point
Sheet 1 ("1. 1A Proforma") is literally a copy of the standalone 1A proforma. The other 14 sheets pull from Sheet 1 via cell references. If you correctly replace Sheet 1's values, the model cascades automatically.

### Rule 3: Preserve Everything When Copying The Template
Load the template with `data_only=False` to preserve all formulas, formatting, charts, named ranges, data validations, and conditional formatting.

### Rule 4: The 1A Layout Has A Row Offset
The standalone 1A file and the 1A tab inside the Reverse 1B have the SAME layout but the Reverse 1B version is shifted down by 1 row. Always verify actual cell positions by reading both files.

### Rule 5: Unit Mix Is The Hard Part
The Birchmount template has 3 unit type rows. Other 1As may have 9+ unit types. Consolidate into the template's structure using weighted averages for SF and rents.

---

## 1A Proforma Layout (Dynamic Parsing)

The proforma has consistent COLUMNS but the ROW positions shift depending on unit count. Parser must find sections dynamically, DO NOT hardcode row numbers.

**Columns are fixed:**
```
D col:  Labels (unit type names, expense names)
E col:  Unit size (SF) / label text
F col:  Unit count / parking spaces / rates
G col:  Unit mix % / monthly fees / expense rates
H col:  Total SF / annual per unit
I col:  Monthly rent per unit / monthly per unit
J col:  $/SF

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [droduque/reverse-1b-tool](https://github.com/droduque/reverse-1b-tool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->

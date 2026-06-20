---
trigger: always_on
description: Use for preparing, reviewing, validating, or debugging Inline XBRL (iXBRL) and XBRL filings. Trigger on iXBRL, XBRL, ESEF, EDGAR/EFM, UK FRC/HMRC, Dutch SBR/KvK/AFM, EBA/EIOPA DPM, IFRS, US-GAAP, EDINET, MCA, Arelle, taxonomy packages, report packages, extension taxonomies, anchoring, block or narrative tagging, fact mapping, contexts, units, decimals, transformation formats, calculation/linkbase/dimension errors, and validator codes such as FR-NL-*, EFM.6.*, ESEF.*, xbrldie:*, xbrldte:*, and xb
---


# iXBRL skill

Inline XBRL embeds XBRL facts inside an XHTML host document via `ix:*`
elements: one file, two audiences (human reader + machine consumer).

This skill provides reference material, scripts, and decision-rules
for iXBRL work across the major regulatory regimes. It does **not**
replace the regulator's filer manual; it routes you to the right page
of the right manual and encodes patterns experts recognise on sight.

## When you load this skill, do this first

1. **Identify the regulator and reporting basis.** The same iXBRL file
   passes or fails depending on which validator runs. Ask the user
   which jurisdiction and which taxonomy. Common combinations:
   - EU listed issuer, IFRS consolidated AFR → **ESEF**, see `references/esef.md`
   - US SEC registrant → **EDGAR / EFM**, see `references/sec-edgar.md`
   - Dutch entity (KvK deposit or AFM listed) → **NL Taxonomie / SBR**, see `references/nl-sbr.md` (and the NL section of `references/taxonomies.md` for entry-point catalogue)
   - UK statutory accounts or HMRC tax → **UK FRC Suite**, see `references/taxonomies.md`
   - Bank or insurer supervisory return → **EBA / EIOPA DPM**, see `references/taxonomies.md`
   - IFRS digital financial statements (no jurisdictional overlay) → **IFRS Accounting Taxonomy**, see `references/taxonomies.md`
2. **Pin the operative rules to the reporting period — bi-temporal.**
   Taxonomies and filing rules are *versioned per year*, and the rules
   in force when a report was prepared are not necessarily the rules in
   force today. Before reviewing or validating, state explicitly:
   - **Which financial year** the report covers (use the period in
     `<xbrli:period>`, not today's date).
   - **Which taxonomy generation and version** applied for that year
     (ESEF 2024 ≠ ESEF 2025; NT19 ≠ NT20; FASB 2024 GRT ≠ 2025 GRT;
     FRC 2025 Suite ≠ 2026 Suite; EBA Reporting Framework 4.2 ≠ 4.4).
   - **Which Filing Rules / Filer Manual edition** applied (ESEF
     Reporting Manual editions, SEC EDGAR Filer Manual volume/version,
     SBR Filing Rules NT-generation supplement).
   Confirm against `references/taxonomies.md` (entry-point catalogue),
   `references/nl-sbr.md` §2 (Dutch bi-temporal cheatsheet), and the
   regulator's published cut-in dates. **Do not apply current-year
   rules retroactively to a prior-year filing** — e.g. KvK Dutch GAAP
   notes block-tagging is a FY2026 obligation, not a FY2024 one, and
   declaring its absence on a FY2024 deposit a defect would itself be
   the defect.
3. **Choose your validation profile.** Use `scripts/validate_with_arelle.sh
   <file> <profile>` (`esef`, `efm`, `ukfrc`, `hmrc`, `core`). Run
   `core` first to isolate XBRL 2.1 violations from jurisdictional ones.
4. **Prepare an Arelle iXBRL Viewer for review.** When reviewing a
   local iXBRL file or document set, generate a viewer with the Arelle
   iXBRL Viewer plugin before doing the content-level review. The
   full preparation command (single file, document set, stub viewer
   mode), version-pinning guidance, and the per-step review checklist
   live in `references/viewer.md`.
5. **Use the live filing corpus for real examples.** For ESEF, UKSEF,
   and Ukraine filings, use <https://filings.xbrl.org/> before and
   after authoring:
   - Filter the index by **Country** (for example `NL` for the
     Netherlands, or another listed country) to inspect filings from the
     relevant market.
   - Open the Inline XBRL viewer to compare how facts, continuations,
     hidden facts, labels, dimensions, and note block tags appear in a
     real report.
   - Download or inspect the xBRL-JSON and XBRL Report Package when you
     need concrete examples of fact values, contexts, units, taxonomy
     package layout, and validation messages.
   - Treat the corpus as evidence, not as authority: the repository is
     not complete, and many included filings have validation errors or
     warnings. Use it to learn market practice, then validate against
     the operative regulator rules.

## How to use the references

Each reference is a focused dive. Load on demand — do **not** read all
of them up front.

| If the question is about… | Read |
|---|---|
| What `ix:nonFraction`, `decimals`, `contextRef`, transformation registry, calc weights mean | `references/spec.md` |
| QNames, SQNames, NCNames, substitution groups, item types (monetary / decimal / shares / pure / textBlock / date / boolean / QName), concept attributes (`periodType`, `balance`, `nillable`) | `references/types.md` |
| DTS, XLink primitives, all five standard linkbases, role / arcrole types, tuples, footnote model vs `ix:footnote`, OIM (xBRL-XML / -JSON / -CSV), versioning, nil-value policy, instance pointers (`schemaRef` / `linkbaseRef` / `roleRef` / `arcroleRef`) | `references/structure.md` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MaxSchoon/ixbrl](https://github.com/MaxSchoon/ixbrl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

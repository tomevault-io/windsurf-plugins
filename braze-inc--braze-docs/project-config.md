---
trigger: always_on
description: Use when analyzing Level Access (ADA) scan exports for Braze Docs, triaging WCAG accessibility findings, or applying critical-severity fixes from the Web ADA Compliance Committee report. Covers filtering to critical only, mapping URLs to repo paths, fixing _docs/ and templates (not _lang/), and generating the Issues report CSV.
---


# Applying ADA accessibility fixes from scan exports

This rule analyzes Level Access scan results for the Braze Docs site. The Web ADA Compliance Committee provides a monthly report; use this file to triage findings and apply fixes.

> **Scope: Critical issues only.** Filter the "equal-access Accessibility" sheet to rows where **Severity** = `critical`. Ignore high, low, and info findings.

## Export format

The scan produces an Excel file (`.xlsx`) with two sheets:

### Sheet: Summary

| Column   | Description                          |
|----------|--------------------------------------|
| URL      | Page URL (e.g., `https://www.braze.com/docs/...`) |
| Page Title | Document title                     |
| Errors   | Count of errors on the page          |
| Warnings | Count of warnings                   |
| Passed   | Count of passed checks               |
| Duration | Scan duration                       |

### Sheet: equal-access Accessibility

| Column           | Description                                      |
|------------------|--------------------------------------------------|
| URL              | Page URL where the issue was found               |
| Page Title/Affected | Page or element affected                     |
| Standard         | WCAG                                             |
| Level            | A, AA, AAA                                       |
| Errors           | Count                                            |
| Severity         | critical, high, low, info                         |
| Guideline/Summary | WCAG guideline (e.g., 4.1.2 Name, Role, Value) |
| Rule Title       | Short rule name                                  |
| Description      | Rule description                                 |
| Rule Id          | Rule identifier (e.g., `input_label_exists`)     |
| Finding URL      | Link to detailed finding (Essentia11y)            |
| Code Snippet     | HTML/element with the issue                      |
| Selector/Xpath   | DOM path to the element                          |
| What is the problem | Plain-language problem description            |
| Why it matters   | Impact for users                                 |
| What to do       | Remediation guidance                             |

---

## Step 1: Filter to critical only

In the "equal-access Accessibility" sheet, include only rows where **Severity** = `critical`. Exclude high, low, and info.

## Step 2: Triage by ownership

Not all findings can be fixed in the docs repo. Categorize each **critical** finding:

| Category        | Criteria | Where to fix | Action |
|-----------------|----------|--------------|--------|
| **Docs content** | Alt text, heading structure, image descriptions in article body | `_docs/**/*.md`, `_includes/**/*.md` | Update markdown in this repo |
| **Docs templates** | Layouts, shared includes (header, nav, search, forms) | `_includes/`, `_layouts/`, `assets/` | Update HTML/Liquid in this repo |
| **_lang/ content** | Localized pages under `_lang/<locale>/` | `_lang/**/*.md` | **Do not edit.** Add to Issues report only |
| **Platform/site** | Third-party widgets, cookie banner, site chrome not in docs repo | Docs site platform (separate repo) | Add to Issues report; escalate |
| **Third-party**   | Search (Algolia), analytics, etc. | External service | Add to Issues report; document for vendor |

**How to tell:** Inspect the **Code Snippet** and **Selector/Xpath**. If it references `su__` (search), `searchForm`, `search-box-autocomplete`, `nav`, `header`, or `form` in shared layout, it may be template/platform or **third-party** — the search widget HTML is injected by external scripts (Swiftype/Algolia) and is not editable in this repo. If it references `img` with missing alt, `div` used as heading, or content inside the main article area, it's likely docs content.

**Docs-repo patterns (fixable here):** `glossary_search`, `api_search`, `partner_search` → `_layouts/api_glossary.html`, `glossary_page.html`, `partner_page.html`, and other glossary/report layouts.

---

## Step 3: Map URL to file path

Convert scan URLs to docs repo paths:

- Base: `https://www.braze.com/docs` → `_docs/`
- Path: `/user_guide/administrative/access_braze/single_sign_on` → `_docs/_user_guide/administrative/access_braze/single_sign_on.md` (or index in `single_sign_on/`)

### Do not edit `_lang/` files

**Never update files under `_lang/`.** Localized content mirrors `_docs/` structure (e.g., `_lang/de/`, `_lang/fr_fr/`). When a finding maps to a path under `_lang/`, do not edit it. Instead, add the issue to the **Issues report** (see below). Fix the corresponding source file in `_docs/` when applicable; the localization team can sync translations separately.

---

## Step 4: Prioritize critical findings

1. **By fixability**: Docs content fixes first (direct edits); template fixes may affect many pages
2. **By rule type** (common critical rules):
   - `input_label_exists`, `aria_accessiblename_exists` → Often template/layout or third-party

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/braze-inc) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->

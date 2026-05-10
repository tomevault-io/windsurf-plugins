---
trigger: always_on
description: The Spring '26 `Blob.toPdf()` rendering engine has strict requirements for image URLs in HTML:
---

# CLAUDE.md — SalesforceDocGen Project Guidelines

## Critical: Blob.toPdf() Image URL Rules

The Spring '26 `Blob.toPdf()` rendering engine has strict requirements for image URLs in HTML:

- **MUST use relative Salesforce paths**: `/sfc/servlet.shepherd/version/download/<ContentVersionId>`
- **NEVER use absolute URLs**: `https://domain.com/sfc/servlet.shepherd/...` — fails silently (no exception, broken image)
- **NEVER use data URIs**: `data:image/png;base64,...` — not supported, renders broken

In `DocGenService.buildPdfImageMap()`, do NOT prepend `URL.getOrgDomainUrl()` to ContentVersion download URLs. Keep them relative. The `Blob.toPdf()` engine resolves relative Salesforce paths internally.

## Critical: Zero-Heap PDF Image Rendering

For PDF output, `{%ImageField}` tags with ContentVersion IDs MUST skip blob loading. The `currentOutputFormat` static variable is set to `'PDF'` before `processXml()` calls. In `buildImageXml()`, when `currentOutputFormat == 'PDF'` and the field value is a ContentVersion ID (`068xxx`), query only `Id, FileExtension` (NOT `VersionData`) and store the relative URL. This is what enables unlimited images in PDFs without heap limits.

**NEVER** add `VersionData` to the SOQL query in the PDF path. Each image blob would consume 100KB-5MB+ of heap, and with multiple images this immediately exceeds governor limits.

## PDF Image Pipeline

### How template images are prepared (on save)

When an admin saves a template version (via `DocGenController.saveTemplate()`), the system calls `DocGenService.extractAndSaveTemplateImages(templateId, versionId)`. This method:

1. Downloads the DOCX/PPTX ZIP from the template's ContentVersion
2. Reads `word/_rels/document.xml.rels` to find all `<Relationship>` entries with `Type` containing `/image`
3. For each image relationship, extracts the image blob from `word/media/`
4. Saves each image as a new ContentVersion with `Title = docgen_tmpl_img_<versionId>_<relId>` and `FirstPublishLocationId = versionId`

This pre-extraction is essential — it creates committed ContentVersion records that `Blob.toPdf()` can reference by relative URL at generation time.

### How template images are rendered (on generate)

At PDF generation time, `buildPdfImageMap()` queries for these pre-committed CVs:
- Finds the active template version
- Queries `ContentVersion WHERE Title LIKE 'docgen_tmpl_img_<versionId>_%'`
- Builds relative URLs: `/sfc/servlet.shepherd/version/download/<cvId>`
- `DocGenHtmlRenderer.convertToHtml()` embeds these as `<img src="/sfc/...">` in the HTML
- `Blob.toPdf()` resolves the relative paths and renders the images

## Package Info

- Package type: Unlocked 2GP with namespace `portwoodglobal`
- Package name: Portwood DocGen
- DevHub: `Portwood Global - Production` (dave@portwoodglobalsolutions.com)
- Dev scratch org: `docgen-test-ux`
- Demo scratch org: `docgen-demo-v2`
- Website: https://portwoodglobalsolutions.com

## Key Architecture

- PDF rendering has two paths in `mergeTemplate()`:
  1. **Pre-decomposed (preferred)**: Loads XML parts from ContentVersions saved during template version creation. Skips ZIP decompression entirely. ~75% heap savings. Used for PDF output when XML CVs exist.
  2. **ZIP path (fallback)**: Full base64 decode + ZIP decompression. Used for DOCX/PPTX output, or PDF when pre-decomposed parts don't exist (older templates not yet re-saved).
- After merge: `buildPdfImageMap()` → `DocGenHtmlRenderer.convertToHtml()` → `Blob.toPdf()` with VF page fallback
- The Spring '26 Release Update "Use the Visualforce PDF Rendering Service for Blob.toPdf() Invocations" is REQUIRED

## Critical: Giant-Query PDF Has THREE Merge-Tag Resolution Paths

`DocGenGiantQueryAssembler` does **not** go through `processXml()`. Tags are resolved in three distinct layers before `Blob.toPdf()`:

1. **Row-level tags (inside the `{#Rel}...{/Rel}` loop)** — rendered per-record by `DocGenService.renderLoopBodyForRecords()` → `processXml()`. Full formatter support (currency, date, locale, aggregates, everything processXml does).
2. **Parent-level tags (outside the loop — headers, titles, summaries)** — resolved by `DocGenGiantQueryAssembler.resolveParentMergeTags()`. Matches `{Field}`, `{Owner.Name}`, and `{Field:format}` forms. Format-suffix tags are routed back through `DocGenService.processXmlForTest()` with a mini 1-field data map so locale/currency/date formatting is reused (v1.51.0+).
3. **Aggregate tags (grand totals across the giant relationship)** — `{SUM:Rel.Field}`, `{COUNT:Rel}`, etc. resolved by `DocGenGiantQueryAssembler.resolveGiantAggregateTags()` via a single SOQL aggregate query, governor-safe for 60K+ rows (v1.50.0+). Field validation is against `Schema.getGlobalDescribe()`, NOT the query config's declared columns — aggregate fields don't have to be rendered columns (v1.52.0+).

**When adding a new merge-tag feature**, decide which of the three paths it belongs to and implement it there — adding to `processXml()` alone won't make it work for giant queries. Missing from all three paths = silent pass-through as literal template text in the PDF.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Portwood-Global-Solutions/DocGen](https://github.com/Portwood-Global-Solutions/DocGen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-28 -->

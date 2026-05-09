---
trigger: always_on
description: Downloading web sources for writing textbook chapters
---



# Web Source Fetching Strategies

This document contains rules and strategies for fetching authoritative content from various web sources. It is a **living document** — add new entries as you discover what works for each website.

---

## Core Principles

1. **Never rely on internal world knowledge for technical content** — every claim must trace to a downloaded source
2. **Source of truth hierarchy**: LaTeX/source code > HTML > PDF > summaries
3. **Download first, read later** — save content locally before processing
4. **Prefer raw sources** — GitHub raw files, arXiv LaTeX, not rendered HTML
5. **`search_web` is for discovery only** — use it to identify sources, not to extract content

---

## Anti-Patterns to Avoid

| ❌ Don't Do This | ✅ Do This Instead |
|-----------------|-------------------|
| Use `search_web` for content | Use it only for source discovery |
| Fabricate URLs | Only use URLs you've actually fetched |
| Regenerate content from memory | Quote/cite downloaded sources |
| Use browser for static pages | Use `curl` or `read_url_content` |
| Link to external URLs for images | Download images locally |
| Conclude a source is incomplete because grep found no matches | List section headings first (`grep '^##'`), then read relevant sections. A grep false negative is not evidence of missing content. |

---

## Extraction Completeness Verification (MANDATORY for Web Sources)

**Readability is necessary but not sufficient.** A web extraction can produce a readable, substantial `.md` file that is *missing entire sections* due to a soft paywall, JavaScript rendering failure, or extraction timeout. This is especially common with Substack (free preview + paywalled body) and Medium.

**After extracting any web article, verify structural completeness:**

1. **List section headings:** `grep '^##\|^###\|^####' content.md` — the output should show a logical article structure (introduction, body sections, conclusion).
2. **Check for paywall markers:** `grep -i 'upgrade to paid\|subscribe to continue\|for paid subscribers\|unlock this post' content.md` — if any of these strings appear, the extraction hit a paywall boundary. Re-extract with `--profile` for the relevant site (e.g., `--profile substack`).
3. **Check the ending:** Read the last 20 lines of the file. Does the article end with a conclusion/summary, or does it cut off abruptly with a subscription prompt? An abrupt ending signals truncation.
4. **Compare against expected content:** If you know the article discusses topics X, Y, and Z (from web search summaries or the TEXTBOOK-PLAN), verify that the section headings include all three topics. If topic Z is missing from the headings, the extraction may be incomplete.

**Do NOT use term-grep with low result limits to assess completeness.** Searching for a specific term (e.g., "ORM") and getting zero results does not mean the content is missing. The term might appear under a different section heading, use different phrasing, or be buried in a section your grep missed due to result limits. Always check section headings first, then read the relevant sections directly.

---

## Decision Tree: Choosing a Fetch Method

```
Is it an arXiv paper?
├─ YES → Download LaTeX source: curl arxiv.org/src/PAPER_ID (ALWAYS prefer this over PDF)
└─ NO
   ├─ Is it a PAYWALLED academic paper? (SAGE, APA, Elsevier, Springer, etc.)
   │  └─ YES → Follow the Paywalled Paper Retrieval Cascade (see section below):
   │           1. Check arXiv for preprint
   │           2. Search author's personal/university website
   │           3. Search Semantic Scholar, PubMed Central, ERIC, CORE
   │           4. Search using unique abstract text + filetype:pdf
   │           5. Check gwern.net archive
   │           6. Try Wayback Machine
   │           7. Flag to user (LAST RESORT)
   │           Then: verify with pdfinfo (page count), extract with mistral_ocr.py
   ├─ Is it a PDF?
   │  ├─ Is it a OneNote export? (single giant page, handwritten ink, no text layer)
   │  │  └─ YES → python scripts/onenote_pdf_to_markdown.py file.pdf -o output/
   │  │           Then feed the extracted PNGs to the IDE's vision LLM (Cursor/Antigravity)
   │  │           Do NOT use Mistral OCR for handwritten OneNote notes (poor accuracy)
   │  └─ NO (standard rendered PDF: textbooks, papers, reports, scanned docs)
   │     ├─ Need high-quality Markdown + images?
   │     │  ├─ YES → python scripts/mistral_ocr.py file.pdf -o output/
   │     │  └─ NO (just text) → pdftotext -layout file.pdf > file.txt
   │     └─ Alternative: marker-pdf (local, slower)
   ├─ Is it a DOCX file?
   │  ├─ YES → pandoc input.docx -o output.md (works well)
   │  └─ NO
   ├─ Is it a GitHub repo, gist, or file?
   │  ├─ Repo/gist → git clone into sources/ (see GitHub section below)
   │  ├─ Single file → curl raw.githubusercontent.com/OWNER/REPO/BRANCH/PATH
   │  └─ NO
   │     ├─ Is it ANY web page (blog, docs, tutorial, login-gated, JS-heavy)?
   │     │  ├─ YES → python scripts/authenticated_extract.py "URL" (DEFAULT for all web content)
   │     │  │        Add --profile NAME for login-gated sites (Substack, Medium)
   │     │  │        Add -s "article" for unknown sites if full-page extraction is noisy

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AI-Learning-Gems/AI-Learning-Gems.github.io](https://github.com/AI-Learning-Gems/AI-Learning-Gems.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->

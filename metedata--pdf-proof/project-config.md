---
trigger: always_on
description: Use this skill when the user needs visual proof that specific values exist in a PDF — not just to read a PDF, but to see exactly where a number, amount, clause, or field appears on the page with a highlighted screenshot. Trigger when someone is cross-referencing a PDF against something else (a form they're filling out, a claim, a conversation, another document) and needs confirmation with evidence. Common signals: entering data into TurboTax or a form and pulling source numbers from last year's 
---


# PDF Proof Page Generator

## What This Skill Does

When a user needs to verify that specific values match their source PDF documents, this skill:

1. Searches the PDF(s) for the exact text of each value
2. Crops a tight, readable section around each match
3. Draws a translucent orange highlight over the found value (using exact text coordinates, not guesswork)
4. Cross-checks by reading text back from the highlighted region to confirm correctness
5. Generates a clean HTML proof page with a summary table, confidence indicators, and per-value proof cards showing the actual screenshots

The output is a standalone HTML file that references cropped PNG screenshots — a visual audit trail the user can keep, share, or refer back to.

**Important:** Always generate the HTML proof page in your first response. Do not just show screenshots inline or summarize values in chat — the shareable proof document is the whole point. Find, extract, and assemble the proof page in one pass.

## How It Works

The key insight is using PyMuPDF's `page.search_for(text)` to get **exact PDF-coordinate rectangles** for each value, then using those coordinates to draw highlights at mathematically precise positions. This avoids the problem of CSS-percentage-based overlays that never quite land in the right spot.

### Step-by-Step Process

#### 1. Identify what needs to be proven

From the user's request, extract:
- **Values to verify**: The specific numbers, amounts, or text the user wants confirmed
- **Source PDFs**: Which PDF documents contain the source data
- **Context for each value**: What form/line/field the value comes from (e.g., "Form 1040, line 15", "Invoice #4521, total amount", "Section 3.2, indemnification cap")

#### 2. Search and locate values in the PDFs

Use the bundled `scripts/extract_proof.py` script. It handles the PyMuPDF text search, cropping, and highlight drawing.

**Step A — Find first.** PDFs often contain the same value in multiple places — a form field and its instructions, a header and a total row, or a cross-reference on another page. Jumping straight to extract risks highlighting the wrong occurrence and producing a misleading proof. Run a find first to see all matches:

```bash
python3 /path/to/skill/scripts/extract_proof.py \
  --pdf "/path/to/source.pdf" \
  --search "3,000" \
  --page 39 \
  --mode find
```

Review the output — if there are multiple matches, note which index corresponds to the actual value vs. mentions in labels, headers, or instructions.

**Step B — Extract with verification:**

```bash
# Single highlight (typical for numbers):
python3 /path/to/skill/scripts/extract_proof.py \
  --pdf "/path/to/source.pdf" \
  --search "1,250.00" \
  --page 5 \
  --output "/path/to/output/proof_total.png" \
  --highlight value \
  --mode verify \
  --json

# Multiple highlights (for policy/text questions):
python3 /path/to/skill/scripts/extract_proof.py \
  --pdf "/path/to/source.pdf" \
  --search "Pets are allowed" "$25 per month" "written permission" \
  --page 3 \
  --output "/path/to/output/proof_pet_policy.png" \
  --highlight value \
  --mode verify \
  --json --context 120
```

Parameters:
- `--pdf`: Path to the source PDF
- `--search`: One or more text strings to find and highlight (multiple terms = multiple highlights on the same screenshot)
- `--page`: 1-indexed page number to search on (if known; omit to search all pages)
- `--context`: How many PDF points of vertical context to include above and below the found text (default: 80). Increase for more surrounding context.
- `--output`: Where to save the cropped PNG
- `--highlight`: Highlight mode — `value` (tight box around the text), `row` (full-width band at the text's vertical position), or `none`
- `--scale`: Render scale factor (default: 3 for high-res)
- `--mode`: `find` (locate text), `extract` (crop+highlight), or `verify` (extract + read text back to cross-check)
- `--prefer`: When multiple matches exist — `right` (rightmost, best for forms), `first`, or `last` (default: `right`)
- `--match-index`: Use the Nth match (0-indexed), overrides `--prefer`
- `--json`: Output structured JSON with confidence and verification results

If you don't know the exact page number, run a search pass first:

```bash
python3 /path/to/skill/scripts/extract_proof.py \
  --pdf "/path/to/source.pdf" \
  --search "1,250.00" \
  --mode find
```

This prints every page and coordinate where the text appears, so you can pick the right one.

#### 3. Interpret confidence and verification results

When using `--mode verify --json`, the output includes:

```json
{
  "confidence": "high",
  "confidence_note": "Single match on page",
  "verification": {
    "readback_text": "1,250.00",
    "text_match": true,
    "status": "pass"
  }
}
```

**Confidence levels (based on verification status, not match method):**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [metedata/pdf-proof](https://github.com/metedata/pdf-proof) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

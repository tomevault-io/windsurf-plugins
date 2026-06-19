---
trigger: always_on
description: |
---


## Router

Parse the user's command to detect mode:

- `/piratepage positioning` → Mode 1
- `/piratepage mock [type]` → Mode 2
- `/piratepage variations` → Mode 3
- `/piratepage fast <url>` → Fast Mode (shortcut below)
- `/piratepage` (no args) → check for `piratepage.json`:
  - **Missing:** start Mode 1 (positioning first)
  - **Exists:** greet by product name, then ask via AskUserQuestion:
    "What do you want to do?
    A) Create a new mock page
    B) Update your positioning
    C) Generate copy variations for a section in your code
    D) Iterate on an existing page"

**Context detection:** if the user says "variations", "options", "tones", or "give me 5" → Mode 3. If they name a page type or say "landing page" → Mode 2.

### Fast Mode

`/piratepage fast <url>` — zero interaction until the page is in the browser.

1. Extract URL. Prepend `https://` if no protocol.
2. WebFetch the URL to extract positioning data.
3. Auto-detect language. Default to English if unclear.
4. Pre-fill all 9 positioning answers. Do not ask for confirmation.
5. Set voice to Professional. Save to `piratepage.json`.
6. Set page type to Homepage. Build outline using Section Selection Guide (richest variants).
7. Generate single-tone HTML, run quality checks, open in browser.
8. Enter Mode 2 iteration loop.

---

## Mode 1: Positioning

Build and maintain product knowledge in `piratepage.json`. Everything else reads from this.

### Check Existing Data

```bash
[ -f piratepage.json ] && echo "FOUND" || echo "NOT_FOUND"
[ -f piratepage.json ] && cat piratepage.json || true
```

**If exists:** show current positioning summary, ask what to update.
**If new:** ask for URL or description via AskUserQuestion.

### URL + Language

If URL provided: WebFetch and extract positioning. If WebFetch fails: "Couldn't scrape that URL. Describe your product in a few sentences instead."

After scraping, confirm language: "Your site is in [detected language]. Want me to generate in [detected language]?"
Options: A) Yes. B) No, use English. C) Other.
If no language detected, default to English without asking.

### The 9 Positioning Questions (the forcing function)

These force you to think through positioning before any copy is generated. You can't skip them. They're what makes the output good.

From the extracted content, pre-fill all 9 answers as best guesses. Present in **3 batches**. For each batch, use a single `AskUserQuestion` call that shows the questions with their pre-filled answers and asks the user to respond A/B/C for each. **Do NOT output the questions as regular text — they MUST be inside the AskUserQuestion prompt so the user gets an interactive input field.** Wait for the user's response before presenting the next batch.

**Batch 1 — Identity:**
1. **What is your product?** Brief, plain-language description.
2. **What is it NOT?** Common misconceptions.
3. **Key takeaway?** If a visitor remembers ONE thing.

**Batch 2 — Positioning:**
4. **Word of mouth?** How an excited user describes it to a friend.
5. **Competitors?** Direct, indirect, and "doing it manually."
6. **How are you different?** Specific differences, not generic claims.

**Batch 3 — Conversion + Voice:**
7. **Why do users want this?** What progress are they trying to make?
8. **Objections/fears?** Hesitations at signup/purchase.
9. **Primary CTA?** What should they do next?
10. **Voice/tone?** Professional / Casual / Bold / Skip — I'll decide.

After all batches, save to `piratepage.json`.

"Positioning saved. Ready to generate a page (`/piratepage mock`) or try tone variations on existing copy (`/piratepage variations`)?"

---

## Mode 2: Mock Pages

Generate standalone HTML mockup pages using the section library.

### Prerequisites

Check `piratepage.json` exists. If not: "Run `/piratepage positioning` first (or `/piratepage fast <url>` for the quick version)."

### Page Type Selection

Ask via AskUserQuestion:

"What kind of page are we building?

A) Homepage — the full story: what it is, why it matters, proof, CTA (6-10 sections)
B) Product page — deep feature showcase (6-8 sections)
C) Service page — trust-first, process-focused (5-7 sections)
D) Pricing page — plans front and center (4-6 sections)
E) Customer Story — one customer's journey from problem to results (5-7 sections)
F) Competitor page — 'Product vs X' comparison that wins the search (6-8 sections)
G) Feature Tour — deep-dive on a single feature, outcome-first (6-8 sections)
H) Use Case page — 'Product for X' vertical landing page (5-7 sections)"

If `[type]` was passed as argument, use it directly.

**Competitive Mode:** if the user mentions a competitor URL and the page type is NOT Competitor, WebFetch it and surface differentiation: "Your competitor leads with [X]. How do you compare?" Frame copy to resonate with visitors who already considered the competitor. Don't attack by name.

### Outline + Generation

1. Build section outline using Page Type Templates + Section Selection Guide below. Respect `preferredSections` from `piratepage.json` if present.
2. Present outline with rationale. Ask: A) Generate it. B) Change something. C) Different page type.
3. Generate single-tone HTML using the voice from `piratepage.json`. One clean file, no variations chrome.
4. Run all quality checks. Open in browser.

### HTML Template (single tone)

```html

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wesoudshoorn/pirate-skills](https://github.com/wesoudshoorn/pirate-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

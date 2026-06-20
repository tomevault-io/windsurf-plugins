---
trigger: always_on
description: Use when user asks to analyze institutional 13F holdings, compare fund manager positions, track quarterly portfolio changes, identify long-term core holdings, or research what major investors are buying/selling. Best suited for retail investors and independent researchers who want readable analysis rather than raw filing data. Triggers on "13F", "holdings", fund manager names, or institutional investment analysis requests.
---


# 13F Institutional Holdings Analysis

## Overview

Analyze SEC 13F filings to track what major institutional investors are buying, selling, and holding. Core capability: fetch raw data from SEC EDGAR, parse holdings, compare across quarters, and produce readable reports for retail investors and independent researchers.

**13F = quarterly snapshot of long positions for US institutions managing >$100M. Filed within 45 days of quarter-end.**

## When to Use

- User asks about institutional holdings or 13F filings
- User names a fund manager and wants portfolio analysis
- User wants to compare holdings across quarters or across managers
- User asks "what is [manager] buying/selling"
- User wants sector/industry allocation analysis
- User asks about a specific stock's institutional ownership history

## Limitations to Always Disclose

1. **45-day lag** — Q2 data appears by ~Aug 15, reflects June 30 snapshot
2. **Long-only** — no shorts, options, futures, or derivatives visible
3. **Confidential treatment** — active positions may be hidden temporarily
4. **Snapshot only** — high-frequency/quant funds may have turned over positions within the quarter

## Language Detection

Detect the user's language from their input. The report should adapt to match:

**English user (default):**
- All UI: tab labels, section headers, badges, insight text — English only
- Stock names: English only (e.g., "ALPHABET INC", "BANK OF AMERICA")
- Footer disclaimer: English only
- No Chinese anywhere in the report

**Chinese user:**
- All UI: tab labels, section headers, badges, insight text — Chinese (with English subtitle where helpful)
- Stock names: localized names plus English ticker/company name where helpful
- Footer disclaimer: bilingual
- Manager profile section: Chinese

Use bilingual or localized labels only when they clearly improve usability for that user.

## Onboarding

When the user's input is vague (e.g., just "13F", "holdings", "show me some funds"), **show the Quick Menu from filers-database.md** — a categorized list of popular managers with numbered options.

If user has a watchlist file (`13f_watchlist.md` in the working directory), mention it: "You have X managers in your watchlist. Want to see the latest updates?"

## Input Recognition

User may provide:
- Fund manager name (e.g., "Buffett", "Druckenmiller")
- Fund/entity name (e.g., "Berkshire Hathaway", "Bridgewater")
- CIK number directly
- A stock ticker for reverse lookup (Mode 3: "Who owns TSLA")
- A number from the Quick Menu
- "my watchlist" — load from watchlist file
- "compare X and Y" — Mode 2
- **A description/requirement** — see Discovery Mode below

**First step:** Map input to CIK number. Check filers-database.md for known filers. If not found, try Discovery Mode or search SEC EDGAR.

## Discovery Mode

When the user doesn't know a specific name but describes what they want, enter Discovery Mode:

**Trigger phrases:**
- "Recommend some managers to follow"
- "Who runs a tech-heavy portfolio"
- "Who invests like Buffett"
- "Which managers are buying China-related stocks"
- "I want concentrated, long-term value investors"
- "Show me smaller managers with strong conviction"
- Any vague description of a style, sector, or preference

**Flow:**

1. **Parse the requirement** — extract keywords: style (concentrated/diversified, value/growth), sector preference (technology/financials/consumer), scale (large/small), geography (China-related/US), etc.

2. **Search in filers-database.md first** — filter the 20 pre-mapped managers by matching criteria. Present as options:

```text
Based on your request for "concentrated value investors", here are strong fits:

  A. Duan Yongping (Himalaya) -- ultra-concentrated, very low turnover
  B. Bill Ackman (Pershing) -- concentrated activist value
  C. Seth Klarman (Baupost) -- selective deep value, highly private
  D. Mohnish Pabrai -- extremely concentrated Buffett-style investor

Which one would you like to see? You can pick multiple (for example "A and D"), or say "add them all to my watchlist".
```

3. **If no match in database** — search SEC EDGAR using EFTS full-text search:
```
GET https://efts.sec.gov/LATEST/search-index?q={keywords}&forms=13F-HR
```
Present discovered filers as options with basic info (entity name, CIK, recent filing date).

4. **User selects** → proceed to Mode 1 (single fund) or add to watchlist

**Key principles:**
- Always give **numbered/lettered options**, don't just dump a list
- Include a one-line hook for each option explaining WHY it matches the user's requirement
- Offer to "add them all to my watchlist" as a quick action
- If the user's requirement is about a specific stock ("who is buying NVDA"), redirect to Mode 3 instead

## Watchlist

Users can maintain a personal watchlist at `13f_watchlist.md` in their working directory.

**Format:**
```markdown
# My 13F Watchlist


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jl3032/13f-analysis](https://github.com/jl3032/13f-analysis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

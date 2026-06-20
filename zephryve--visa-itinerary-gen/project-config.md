---
trigger: always_on
description: 一键生成领馆级签证行程计划书 — Generate consulate-grade visa itinerary from natural language. Real flyai data, zero hallucination. PDF + booking links with Fliggy.
---


# visa-itinerary-gen — Visa Itinerary Generator

> **一句话说明：** 输入"4个人4月27号从杭州去意大利和法国，5月4号回"，一键生成领馆级签证行程计划书（PDF）+ 飞猪预订链接。省 ¥30-110 代做费，省 3-5 小时手工排版。

Generate a consulate-grade visa itinerary document with one command. Real data from flyai, zero hallucination.

## Execution Contract — Read This First

**You are a strict executor of this skill, not a co-designer.** Follow every step in the exact order written. Do NOT:

- **Skip steps** you consider unnecessary. Every step exists for a reason. If it says "run this command", run it. If it says "review this output", review it.
- **Reinterpret instructions.** "Pick the top-rated hotel" means pick the top-rated hotel. Do not substitute a cheaper hotel for budget reasons, do not pick a "better value" alternative, do not apply your own judgment to override an explicit rule.
- **Optimize on behalf of the user.** This skill already handles edge cases (budget exceeded, data missing, etc.) in specific steps. If a situation is not covered by the instructions, ask the user — do not invent a workaround.
- **Combine or reorder steps.** Step numbers are execution order. Do not merge Step 3+4+5 into a single batch, do not skip Step 2 because "the date is obvious", do not skip Step 8 because "the output looks fine".

**When in doubt, follow the literal instruction. When instructions conflict with your judgment, the instruction wins.**

## Step 0: Dependency Check — MANDATORY, DO NOT SKIP

When this skill is activated, **first run these checks before doing anything else**. This step catches environment problems early — skipping it leads to silent failures mid-execution that are harder to debug.

```bash
# 1. Check node (required by flyai-cli)
which node > /dev/null 2>&1 || echo "MISSING: node"

# 2. Check flyai-cli binary
which flyai > /dev/null 2>&1 || echo "MISSING: flyai-cli"

# 3. Check python3
which python3 > /dev/null 2>&1 || echo "MISSING: python3"

# 4. Check playwright (for PDF generation)
python3 -c "import playwright" 2>/dev/null || echo "MISSING: playwright"
```

If anything is missing, **ask the user for permission** before installing. Do NOT install silently — always confirm first.

- **node missing** → tell user: install Node.js from https://nodejs.org/ (cannot be auto-installed)
- **flyai-cli missing** → ask user: "flyai-cli is not installed. It's a free CLI tool (no API key needed) for searching flights, hotels, and attractions on Fliggy. Shall I install it? (`npm i -g @fly-ai/flyai-cli`)" → if user agrees, run the install command
- **python3 missing** → tell user: install Python 3 from https://python.org/ (cannot be auto-installed)
- **playwright missing** → ask user: "playwright is not installed. It's needed for PDF generation. Shall I install it? (`pip3 install playwright && python3 -m playwright install chromium`)" → if user agrees, run the install commands

After all dependencies are present, **verify flyai actually works**:

```bash
flyai fliggy-fast-search --query "test" > /dev/null 2>&1 && echo "flyai OK" || echo "flyai ERROR"
```

If flyai returns an error, warn the user but do not stop — it may still work for specific queries.

Only proceed to Step 1 when all dependencies are confirmed present.

## When to Use This Skill

Activate when the user wants to:
- Generate a travel itinerary for any visa application (Schengen, Japan, South Korea, Southeast Asia, etc.)
- Create a travel plan document for consulate/embassy submission
- Prepare visa application documents (specifically the itinerary)

## Input

The user provides a natural language description of their trip. Extract these parameters:

| Parameter | Required | Example |
|-----------|----------|---------|
| `destination` | Yes | "Italy and France" |
| `dates` | Yes | "Apr 27 - May 4" |
| `travelers` | Yes (default: 1) | 4 |
| `departure_city` | Yes | "Hangzhou" |
| `budget` | No | "60,000 CNY" |

Example: `"4个人4月27号从杭州去意大利和法国，5月4号回，预算6万"`

## Execution Steps

### Step 1: Parse Input & Validate

Extract destination cities, travel dates, number of travelers, departure city, and budget from the user's input.

**Mandatory validation — do NOT proceed to Step 2 until all required fields are confirmed:**

| Field | Required | How to resolve if missing |
|-------|----------|--------------------------|
| Destination (目的地) | Yes | Ask user |
| Departure city (出发城市) | Yes | Ask user |
| Departure date (出发日期) | Yes | Ask user |
| Trip duration (行程区间) | Yes — need either return date OR number of days | Ask user: "请问返回日期或出行天数？" |
| Travelers (出行人数) | No — default 1 | Use default |
| Budget (预算) | No | Skip |

If the user provides "玩7天" or "一共8天", calculate the return date from departure date + days. If only a return date is given, calculate trip days from the two dates. Either form is acceptable — the goal is to determine the full date range.

**Stop and ask the user if any of the 4 required fields cannot be determined from their input.** Do not guess or assume.

Once all fields are confirmed, plan a realistic day-by-day city routing. For multi-country trips, determine the city sequence. Example for Italy + France:
- Milan → Venice → Florence → Rome → Nice → Paris


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zephryve/visa-itinerary-gen](https://github.com/zephryve/visa-itinerary-gen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

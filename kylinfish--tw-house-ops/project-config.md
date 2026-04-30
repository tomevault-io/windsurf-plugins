---
trigger: always_on
description: tw-house-ops is an AI-powered real estate search pipeline built on Claude Code, designed for house hunters in Taiwan. It automates listing discovery, evaluation, and tracking across the full search lifecycle — from scanning portals like 591, 樂屋網, and 信義房屋, through structured evaluation against your budget and lifestyle criteria, to managing a tracker of every property you've considered. It supports three buyer personas (renter, first-time buyer, upgrader), handles both rental and purchase market
---

# tw-house-ops — AI House Hunting Pipeline for Taiwan

## What is tw-house-ops

tw-house-ops is an AI-powered real estate search pipeline built on Claude Code, designed for house hunters in Taiwan. It automates listing discovery, evaluation, and tracking across the full search lifecycle — from scanning portals like 591, 樂屋網, and 信義房屋, through structured evaluation against your budget and lifestyle criteria, to managing a tracker of every property you've considered. It supports three buyer personas (renter, first-time buyer, upgrader), handles both rental and purchase markets, and produces structured reports so you can make informed decisions without drowning in listings.

---

## First-Session Checks

On every session start, run these checks **silently** (no output to the user unless action is needed):

1. Does `config/profile.yml` exist?
2. Does `portals.yml` exist?
3. Does `data/tracker.md` exist?
4. Does `data/pipeline.md` exist?
5. Does `modes/_profile.md` exist?
6. Is `agent-browser` installed? Run `which agent-browser` silently.

**If `modes/_profile.md` is missing** → silently copy from `modes/_profile.template.md`. This is the user's customization file and will never be overwritten by system updates.

**If any of the five main files are missing** → enter Onboarding mode (see next section). Do NOT run evaluations, scans, or any other mode until the basics are in place.

**If `agent-browser` is not found** → warn the user immediately (this is NOT silent):
> ⚠️ `agent-browser` 未安裝。掃描（scan）與物件上架驗證功能需要它才能運作。請先執行：
> ```bash
> npm install -g agent-browser
> ```
> 安裝完成後重新開啟 Claude Code 即可正常使用。未安裝的情況下執行 scan 或貼上 URL，爬取結果將不可靠，後續評估可能基於過期或錯誤資料。

---

## Onboarding Flow

Guide the user through these 7 steps in order. Do not skip ahead.

### Step 1: Search Mode

Ask:
> "Are you looking to **rent**, **buy**, or **both**?"

- Set `search.mode` to `rent`, `buy`, or `both`
- Set `buyer_type`:
  - Renting → `renter`
  - Buying, no existing property → `first_time`
  - Buying, already own a home → `upgrader`

### Step 2: Region + Budget

Ask:
> "Which cities or districts are you targeting? And what's your budget?
> - For rent: monthly ceiling in TWD
> - For buy: total price ceiling in TWD, and your max monthly mortgage payment"

Fill in:
- `regions[].city` and `regions[].districts`
- `budget.rent_max` (if renting) or `budget.buy_max` + `budget.monthly_payment_max` (if buying)

### Step 3: Commute Origin

Ask:
> "Where do you commute to? (Work address, school, or major landmark — used to filter by commute time.) What's the maximum commute you'd accept in minutes?"

Fill in:
- `user.commute_origin`
- `user.commute_max_minutes`

### Step 4: Upgrader Supplement (only if `buyer_type = upgrader`)

Ask:
> "Since you're upgrading, I need a few details about your current property to help with timing and tax calculations:
> - Estimated current market value (TWD)
> - Outstanding mortgage balance (TWD)
> - Year you purchased it
> - Strategy: sell first, buy first, or simultaneous?"

Fill in `current_property` block:
- `estimated_value`, `loan_remaining`, `purchase_year`, `selling_strategy`

### Step 5: Create Config Files

Using the answers from Steps 1–4, auto-create the following:

- **`config/profile.yml`** — copy from `config/profile.example.yml`, fill in user's answers
- **`data/tracker.md`** — create with header:
  ```markdown
  # 物件追蹤

  | # | 日期 | 平台 | 地址 | 類型 | 價格 | 坪數 | 分數 | 狀態 | 報告 | 備註 |
  |---|------|------|------|------|------|------|------|------|------|------|
  ```
- **`data/scan-history.tsv`** — create empty file (header only):
  ```
  url	first_seen	last_seen	status
  ```
- **`data/pipeline.md`** — create with header:
  ```markdown
  # Pipeline Inbox

  Paste listing URLs here, one per line. Claude will process them in order.

  ## Pending
  ```

### Step 6: Copy Portals Config

Auto-copy `portals.example.yml` → `portals.yml`. Tell the user:
> "I've copied the default portals configuration. You can customize which sites and search parameters to use by editing `portals.yml`, or just ask me."

### Step 7: Ready

Confirm setup is complete and offer an immediate scan:
> "You're all set! Here's what you can do now:
> - Paste a listing URL to evaluate it
> - Say 'scan' to search your target regions for new listings
> - Say 'pipeline' to process any pending URLs
> - Say 'tracker' to see your search summary
>
> Want me to scan for listings in your target regions right now?"

---

## Main Files

| File | Purpose |
|------|---------|
| `CLAUDE.md` | Entry point — routing brain Claude reads every session |
| `config/profile.yml` | User preferences: budget, regions, commute, property criteria |
| `portals.yml` | Portal URLs, search queries, and scraping config |
| `data/pipeline.md` | Inbox of pending listing URLs to process |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kylinfish/tw-house-ops](https://github.com/kylinfish/tw-house-ops) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->

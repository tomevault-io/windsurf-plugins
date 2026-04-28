---
trigger: always_on
description: Next.js 16.1.1 app with two systems:
---

# Whale-Class / Montree - Developer Brain

## Project Overview
Next.js 16.1.1 app with two systems:
- **Whale Class** (`/admin/*`) - Admin tools (card generators, description review, etc.)
- **Montree** (`/montree/*`) - Real SaaS multi-tenant Montessori school management

Production: `https://montree.xyz` (migrated from teacherpotato.xyz — old domain returns 405 on API calls)
Deploy: Railway auto-deploys on push to `main`
Git remote: `git@github.com:Tredoux555/whale-class.git` (SSH — Cowork VM key "Cowork VM Feb 15" added Feb 15, 2026; old "Cowork VM" Feb 11 key is stale)
Local path: `/Users/tredouxwillemse/Desktop/Master Brain/ACTIVE/whale` (note space in "Master Brain")
**⚠️ Git Push — ALWAYS use Desktop Commander FIRST:** `mcp__Desktop_Commander__start_process` with command `cd ~/Desktop/Master\ Brain/ACTIVE/whale && git push origin main 2>&1` and `timeout_ms: 30000`. Do NOT try Cowork VM SSH keys, GitHub PATs, or `scripts/push-to-github.py` — Desktop Commander on the user's Mac is the only reliable push method.

---

## 📧 GMASS OUTREACH CAMPAIGN — "Montree" (cold email to schools)

**TL;DR:** Outreach is run via **GMass** (Gmail mail-merge extension), driven off the user's spreadsheet **`Montree_Global_Outreach.xlsx`** (also lives as a Google Sheet that GMass syncs Status/DateSent back to). NOT sent from the whale codebase. The super-admin "outreach-campaign" page in the repo is a template builder only — `Mark as Sent` writes to localStorage, never DB. Ignore it.

**The list:** 420 schools across 7 batches — Asia 61, Europe 86, Middle East 56, South Asia 80, Africa 38, Americas 71, Oceania 28. Columns: SchoolName, Email, Country, Region, Website, ContactPerson, Accreditation, AgeRange, DateSent, Status, FollowUp1Sent, FollowUp2Sent, Notes, Batch. As of Apr 7, 2026 only 8 marked Sent in the xlsx (53 blank, 359 Pending) — but GMass campaign report shows 201+ opens already from the Mar 28 send, so the spreadsheet is out of sync with reality. Always re-pull from GMass dashboard / Google Sheet, don't trust the local xlsx.

**Why "it stopped at 50":** Free Gmail (`tredoux555@gmail.com`, gmail.com not Workspace) is hard-capped by Gmail at ~500/day, and GMass self-throttles to ~50/day on free Gmail accounts to protect deliverability. The campaign isn't broken — it's pacing. To "send the rest" the user needs to either (a) resume/extend the campaign in GMass over multiple days, or (b) upgrade to Google Workspace for the higher cap. The "paid Google account" the user mentions is likely paid GMass, not paid Workspace — verify before assuming higher caps.

**GMass campaign details — CORRECTED Apr 7 from dashboard:**
There are **4 separate "Montree" campaigns** all sent on 2026-03-28, NOT one. Totals across all four = **64 schools sent** (not 50). The big one had a 34.7% bounce rate which is the single biggest deliverability risk for the next send.

| Campaign ID | Recipients | Opens | Bounces | Replies |
|---|---|---|---|---|
| 50490844 | 9 | 6 (66.7%) | 1 | 0 |
| 50489320 | **49** | 16 (32.7%) | **17 (34.7%)** | 1 |
| 50489098 | 3 | 1 | 1 | 0 |
| 50488830 | 3 | 2 | 0 | 0 |

- Spreadsheet name in GMass: `Montree Global Outreach → Outreach List` (lives in user's Google Drive — Drive MCP can't see Sheets, only Docs/folders, so locate via Chrome on `drive.google.com`)
- Reports come from `notify@gmass.co` to `tredoux555@gmail.com`
- Subject line: `Montree` (follow-ups become `Re: Montree`)
- Two automatic follow-ups already configured: F1 ~5 days after original ("I wanted to make sure my previous email…"), F2 ~5 days after F1 ("I understand how busy things can get…")
- Each campaign was a separate batch/chunk — not a "single resumable campaign." To send the rest, **build a NEW GMass campaign**, do NOT try to "resume" 50489320.

**🚨 CAMPAIGN ORDER REVERSED Apr 10, 2026 — JOB APPLICATION FIRST, MONTREE PITCH SECOND 🚨**

Strategic reversal Apr 10: **Job application email sends FIRST (starting Fri Apr 10), Montree pitch POSTPONED to ~Apr 27 as follow-up.** Rationale: personal "hire me" email is warmer/shorter, gets principals curious, then Montree pitch lands 2 weeks later for non-responders as the product follow-up. Resume deliberately NOT attached — no asymmetric risk from naming current school. Interested principals will ask for details privately.

**Campaign C — JOB APPLICATION — ☠️ DEAD, SENT BLANK (Apr 10, 2026):**
- Campaign ID: `50686495` / Schedule ID: `51331920`
- Draft ID: `r614453887712204887` (Gmail message ID `19d761413fb0a98d`) — **TRASHED by user**
- **WHAT HAPPENED**: Session 12 attempted to automate GMass via Chrome DOM manipulation. Body was set via `createElement`/`appendChild` which rendered visually but DID NOT update Gmail's internal draft state. GMass read the empty internal state and sent **335 blank emails** to the entire 345-school list at once (no throttle — DOM-set speed settings also didn't persist). ~74 bounced/blocked (54 "Address not found" + 10 "Message blocked" + 10 blocks), so ~261 schools received an empty email with subject "Montessori Teacher & Builder" and no body.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Tredoux555) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-16 -->

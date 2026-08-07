---
trigger: always_on
description: Project-level instructions Claude reads automatically before acting.
---

# YÀN Atelier · Claude Operating Notes

Project-level instructions Claude reads automatically before acting.

## Session continuity protocol (READ FIRST)

This project's memory is shared via directory junctions across 4 CWDs: `D:\YAN_Atelier 品牌`, `D:\YAN_Atelier_Site`, `D:\wechat-claude`, `D:\YAN_AutoEdit`. All four resolve to the same `MEMORY.md` + `project_session_state_*` files. Anything you write to memory in one CWD is visible from the others.

**On session start** (first user message in this project):
1. The auto-memory MEMORY.md is already in your context. Scan for the LATEST `project_session_state_YYYY-MM-DD*` entry — that file is the canonical resume anchor.
2. If user's first message is `继续 YÀN` / `继续上次` / `接 X` / `恢复` / `restart` / `where were we`: read that latest session_state file in full BEFORE replying. Summarize in ≤3 lines: (a) last commit + state, (b) what was in progress, (c) any pending user actions / blockers. Then ask "继续 X 还是新任务?"
3. If user's first message is a fresh task: do NOT volunteer resume context — just work. Only surface a 1-line `(resuming from: <state-file-name>)` hint if the new task touches the same lane as the last session_state.

**Mid-session snapshot triggers** — write/update today's `project_session_state_YYYY-MM-DD.md` memory file when ANY of these fire:
- User says 收工 / 明天继续 / 我先去X / 拜 / bye / 睡了 / 先停一下
- You're about to ship a load-bearing change (commit + push) involving 3+ files OR Hero/H1/pricing/section-structure
- A multi-agent review (CEO/Investor/persona panel) just completed
- User explicitly says 存一下 / 记一下 / save state
- Heap risk: this session has run > ~6 hours of dialog (Claude Code Bun heap caps at ~1.4GB → OOM. See [[user-yan-atelier]] crash 2026-06-23)

**Snapshot format** — body structure: 3 short paragraphs in this order:
1. **Last commit** (SHA + 1-line) + **live URL state** (deployed yes/no)
2. **What was just done** this session (3-6 bullets, factual)
3. **Pending** (user-action items + Claude-action items, separated)

Title slug: `project_session_state_YYYY-MM-DD[_eod|_late|_early].md`. Always add a `MEMORY.md` index line. If today's state file already exists, EDIT it (don't create _v2 / _2 / etc — overwrite the body).

**Stale state supersession** — when a new session_state supersedes an older one (e.g., morning anchor → EOD anchor), add `**SUPERSEDED BY** [[new-slug]]` at the top of the old file's body. Don't delete.

**Daily restart hygiene** — Claude Code's Bun runtime OOMs around 1.4GB heap. Long sessions accumulate transcript in RAM. Tell the user once per restart cycle: "建议每天滚一次 claude.exe; 重开用 `claude --continue --dangerously-skip-permissions` 接上一段对话(用户标准开机口令,见 [[user-daily-boot-command]]); 长上下文用 [[project-session-state-YYYY-MM-DD]] 恢复战略上下文."

## Live URLs

- **Production (LIVE)**: `https://yan-atelier-site.pages.dev/`
  - This is the URL to give the user for any "test the site" / "mobile preview" / "send to customer" request.
  - Mobile + desktop both responsive. Works on any network (no LAN required).
  - Latest commit auto-deploys via Cloudflare Pages ~30-60s after `git push origin main`.
- **Custom domain (NOT yet wired)**: `https://yan-atelier.com/` — DNS not resolving. Do NOT give this URL to user; it returns nothing.

## Mobile testing

When user asks "给我手机测试链接" or "mobile link" or similar:
- **DO**: reply with `https://yan-atelier-site.pages.dev/`
- **DO NOT**: start a local dev server (no dev server needed; site is single-file static HTML).
- **DO NOT**: invent LAN IP addresses like `http://192.168.x.x:3000` — these almost never work and confuse the user.

## Deploy flow

This IS a git repo (despite environment occasionally reporting otherwise). Repo on GitHub → Cloudflare Pages auto-builds on push to `main`.

To ship a change:
1. Edit files in `D:\YAN_Atelier_Site\`
2. `git add <file>` (be specific — avoid `git add -A`; there are untracked WeChat-export images that should NOT be committed)
3. `git commit -m "..."` (include `Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>`)
4. `git push origin main`
5. Wait 30-60s, Cloudflare deploys
6. Hard-refresh on `https://yan-atelier-site.pages.dev/` to verify (browser cache can mask new deploys)

## Operating rules (load-bearing — see BRAND.md for full)

- **R1**: Section titles name the JOB, never enumerate contents/inventory/taxonomy
- **R2 (ZONE)**: Voice by zone — Cold (Home above-fold) strict commerce, Warm (Home below-fold) atelier-register, Deep (/craft, /about, /consult) atelier-encouraged
- **R3**: PDP trust info inline near CTA, never as top promo bar

Read `BRAND.md` for full rule details + worked examples + banned phrasings before any voice/title/PDP changes.

## Substance protection (do NOT touch)

- plique-à-jour technique name + Renaissance lineage (in Plique section)
- Hu Yan name + Founder status (biographical references in About / Maker)
- Yunnan workshop / 滇韵珐琅手工坊 / Yunnan Studio
- 800°C high-fire fact
- Craft page Schema.org Article markup + 10 academic citations
- Hero copy "Wear it. People will ask where you found it. / 戴出去, 别人会问你哪儿淘的。"
- CN view full readability (cascade flips must preserve CN view)

## What NOT to do

- Do not start local dev servers for "mobile testing" — use prod URL
- Do not invent LAN URLs

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jianyouren/yan-atelier-site](https://github.com/jianyouren/yan-atelier-site) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->

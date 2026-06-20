---
trigger: always_on
description: Search, read, analyze, and automate Xiaohongshu (小红书) content via CLI
---


# Redbook — Xiaohongshu CLI

Use the `redbook` CLI to search notes, read content, analyze creators, automate engagement, and research topics on Xiaohongshu (小红书/RED).

**OpenClaw users:** Install via `clawhub install redbook` or `npm install -g @lucasygu/redbook`.

> **⚠️ Research discipline (read this first).** XHS 风控 throttles *reading*, not just writing. Any research that reads more than a handful of notes MUST run through the **[Research Loop](#research-loop--rate-limit-safe-long-running-research)** — **human-paced by default** (~20 s/note, one at a time; a typical job finishes in tens of minutes, large ones spread across the day). Hammering `read` / `comments` / `analyze-viral` in a tight loop trips captcha / IP block (300012) within a few dozen hits and degrades the account for hours. ⚡ Fast mode is **emergency-only** and requires a printed warning + explicit user opt-in. Never fire reads in parallel or with zero delay.

## Usage

```
/redbook search "AI编程"              # Search notes
/redbook read <url>                   # Read a note
/redbook user <userId>                # Creator profile
/redbook analyze <userId>             # Full creator analysis (profile + posts)
```

## Quick Reference

| Intent | Command |
|--------|---------|
| Search notes | `redbook search "keyword" --json` |
| Read a note | `redbook read <url> --json` |
| Get comments | `redbook comments <url> --json --all` |
| Creator profile | `redbook user <userId> --json` |
| Creator's posts | `redbook user-posts <userId> --json` |
| Browse feed | `redbook feed --json` |
| Search hashtags | `redbook topics "keyword" --json` |
| Analyze viral note | `redbook analyze-viral <url> --json` |
| Extract content template | `redbook viral-template <url1> <url2> --json` |
| Post a comment | `redbook comment <url> --content "text"` |
| Reply to comment | `redbook reply <url> --comment-id <id> --content "text"` |
| Batch reply (preview) | `redbook batch-reply <url> --strategy questions --dry-run` |
| Like a note | `redbook like <url>` |
| Unlike a note | `redbook like <url> --undo` |
| List favorites | `redbook favorites --json` or `redbook favorites <userId> --json` |
| Collect a note | `redbook collect <url>` |
| Remove from collection | `redbook uncollect <url>` |
| List followers | `redbook followers <userId> --json` |
| List following | `redbook following <userId> --json` |
| Delete own note | `redbook delete <url>` |
| Check note health | `redbook health --json` or `redbook health --all --json` |
| List user boards | `redbook boards` or `redbook boards <userId> --json` |
| List album notes | `redbook board <board-url>` or `redbook board <boardId> --json` |
| Render markdown to cards | `redbook render content.md --style xiaohongshu` |
| Publish image note | `redbook post --title "..." --body "..." --images img.jpg` |
| Check connection | `redbook whoami` |

**Always add `--json`** when parsing output programmatically. Without it, output is human-formatted text.

---

## XHS Platform Signals

XHS is not Twitter or Instagram. These platform-specific engagement ratios reveal content type and audience behavior.

### Collect/Like Ratio (`collected_count / liked_count`)

XHS's "collect" (收藏) is a save-for-later mechanic — users build personal reference libraries. This ratio is the strongest signal of content utility.

| Ratio | Classification | Meaning |
|-------|---------------|---------|
| >40% | 工具型 (Reference) | Tutorial, checklist, template — users bookmark for reuse |
| 20–40% | 认知型 (Insight) | Thought-provoking but not saved for later |
| <20% | 娱乐型 (Entertainment) | Consumed and forgotten — engagement is passive |

### Comment/Like Ratio (`comment_count / liked_count`)

Measures how much a note triggers conversation.

| Ratio | Classification | Meaning |
|-------|---------------|---------|
| >15% | 讨论型 (Discussion) | Debate, sharing experiences, asking questions |
| 5–15% | 正常互动 (Normal) | Typical engagement pattern |
| <5% | 围观型 (Passive) | Users like but don't engage further |

### Share/Like Ratio (`share_count / liked_count`)

Measures social currency — whether users share to signal identity or help others.

| Ratio | Meaning |
|-------|---------|
| >10% | 社交货币 — people share to signal taste, identity, or help friends |
| <10% | Content consumed individually, not forwarded |

### Search Sort Semantics

| Sort | What It Reveals |
|------|----------------|
| `--sort popular` | Proven ceiling — the best a keyword can do |
| `--sort latest` | Content velocity — how much is being posted now |
| `--sort general` | Algorithm-weighted blend (default) |

### Content Form Dynamics

| Form | Tendency |
|------|----------|
| 图文 (image-text, `type: "normal"`) | Higher collect rate — users save reference content |
| 视频 (video, `type: "video"`) | Higher like rate — easier to consume passively |

---

## Analysis Modules

Each module is a composable building block. Combine them for different analysis depths.

### Module A: Keyword Engagement Matrix

**Answers:** Which keywords have the highest engagement ceiling? Which are saturated vs. underserved?

**Commands:**
```bash
redbook search "keyword1" --sort popular --json
redbook search "keyword2" --sort popular --json
# Repeat for each keyword in your list
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lucasygu/redbook](https://github.com/lucasygu/redbook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

---
trigger: always_on
description: This file briefs Claude Code when working **inside this repo** (adding new skills, editing existing ones, fixing docs). It's not for end-users — they install via the methods in [docs/install.md](./docs/install.md).
---

# CLAUDE.md — repo conventions for Claude Code

This file briefs Claude Code when working **inside this repo** (adding new skills, editing existing ones, fixing docs). It's not for end-users — they install via the methods in [docs/install.md](./docs/install.md).

## What this repo is

~22 [Agent Skills](https://agentskills.io/specification) that wrap the [Longbridge Securities](https://longbridge.com) platform — quotes, charts, fundamentals, valuation, news, watchlist, account analytics, etc. Multilingual triggers (Simplified Chinese / Traditional Chinese / English). The default style is **prompt-only** (SKILL.md tells the LLM what `longbridge ...` command to run); `scripts/` and `commands/` subfolders are allowed but should be opt-in for clear runtime needs (DOCX generation, chart helpers, slash commands), not as a wrapper-by-default.

## Layout

```
longbridge-skills/
├── .claude-plugin/marketplace.json    # plugin marketplace entry
├── skills/                            # ~22 skill folders
│   ├── <slug>/
│   │   ├── SKILL.md                   # required
│   │   ├── references/                # optional — on-demand detail loaded by the LLM
│   │   ├── scripts/                   # optional — Python helpers (e.g. DOCX, charts) when there's a clear runtime need
│   │   └── commands/                  # optional — Claude Code slash commands (`/<command>`)
│   └── ...
├── docs/
│   ├── architecture.md                # multilingual + CLI/MCP routing design
│   └── install.md                     # install / verify / FAQ
├── CLAUDE.md                          # this file
├── README.md
├── LICENSE                            # MIT
└── .gitignore
```

## Conventions for adding or editing a skill

### 1. Slug + directory

- Directory name must match `^[a-z0-9]+(-[a-z0-9]+)*$` (lowercase ASCII + hyphens).
- It MUST equal the `name:` value in the SKILL.md frontmatter.
- Existing skills are namespaced `longbridge-*` (e.g. `longbridge-quote`). The single base skill is just `longbridge`.
- **Slugs are immutable once published.** Never rename or delete a published skill's slug. Installers (`npx skills update`) match by slug, so a rename does not upgrade the old install — it orphans it on every user's machine (stale + competing for the same triggers) while the new name is never picked up by `update`. Reorganize by adding/editing content under the existing slug. If a new slug is genuinely unavoidable, it is a **breaking change**: note it in the release notes and direct users to the full reinstall (`scripts/update.sh`), the only thing that clears orphaned slugs.

### 2. Frontmatter

Required: `name`, `description`. Recommended: `license: MIT`, `metadata`.

```yaml
---
name: longbridge-foo
description: |
  One-paragraph what + when. Triggers: "<zh-Hans keyword>", "<zh-Hant keyword>", "<English keyword>", ...
license: MIT
metadata:
  author: longbridge
  version: "1.0.0"
  risk_level: read_only # or account_read | mutating
  requires_login: false # or true
  default_install: true # or false (mutating skills sometimes opt out)
  requires_mcp: false # true for analysis-tier skills
  tier: read # or analysis (informational)
---
```

`description` must be **≤ 1024 characters total**. The `Triggers:` list inside it is the **only** thing Claude Code uses to decide whether to activate the skill — pack it with multilingual keywords (see §3).

### 3. Language coverage for triggers

`description` MUST cover **Simplified Chinese + English**. Traditional Chinese is only added where the glyph differs from Simplified (e.g. `股价`→`股價`, `汇率`→`匯率`, `经纪`→`經紀`). Identical glyphs appear once only.

- **Simplified Chinese**: 现在多少钱 / 涨跌幅 / 持仓 / ...
- **English**: stock price / current quote / my holdings / ...
- **Traditional Chinese** (divergent glyphs only): 股價 / 漲跌幅 / 持倉 / ...
- **Ticker examples**: NVDA.US, 700.HK, 600519.SH

This reduces description size by ~15–20%, freeing space for more trigger concepts within the 1024-character limit.

### 4. Body — "Response language" directive (mandatory)

Every SKILL.md must include this line right after the H1 + intro paragraph:

```markdown
> **Response language**: match the user's input language —
> English / Simplified Chinese / Traditional Chinese.
> **RULE: Response language priority**: English is the default when language is ambiguous. If the user input is only a slash command, command name, ticker / symbol, or contains no natural-language language signal, you MUST respond in English. Do not infer Chinese from trigger keywords, skill metadata, or examples.
```

This instructs the LLM to detect the user's input language and reply in the same language. English must be listed first and is the mandatory fallback for ambiguous input. Slash-only commands such as `/longbridge-portfolio` default to English because they carry no natural-language signal; the model must not infer Chinese from the multilingual trigger list. Field tables and error reply tables must be 3-column (Simplified / Traditional / English).

### 4b. Body — "Data-source policy" directive (mandatory)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [longbridge/skills](https://github.com/longbridge/skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->

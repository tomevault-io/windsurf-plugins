---
trigger: always_on
description: **Author:** Arseniy Kamyshev (nafigator@gmail.com)
---

# ru-text — Claude Code plugin for Russian text quality

**Author:** Arseniy Kamyshev (nafigator@gmail.com)
**Repo:** https://github.com/talkstream/ru-text
**Site:** https://ru-text.org
**Sponsors:** https://github.com/sponsors/talkstream
**Version:** 2.7.0 | **License:** MIT | **Platforms:** Claude Code, GitHub Copilot, Windsurf, Cursor, Cline, JetBrains (Junie), Continue.dev, Codex CLI, Gemini CLI, Google Antigravity, OpenClaw, Notion

Over 2,000 independently formulated linguistic atoms across 7 thematic areas. No verbatim quotes, full source attribution.

## Priorities

1. **Quality** — every rule must be accurate and actionable
2. **User care** — plugin helps, never restricts. User's explicit style request overrides defaults
3. **Performance** — SKILL.md at most **650 words** (measured `LC_ALL=C wc -w`); references load on demand, not at startup. The figure used to read «under 600» and was false: the file was 615 words when nobody checked, and the always-on dash fix of 01.08.2026 took it to 640. The budget is now a gate in `tools/check-dogfood.sh`, not a sentence — a size claim about the one file loaded on every turn is exactly the kind that goes stale in silence.
4. **Legal safety** — no "distilled from", no implied endorsement, no author names in section headers

## Architecture

```
.claude-plugin/plugin.json      → Claude Code plugin metadata
.codex-plugin/plugin.json       → Codex CLI plugin metadata
.cursor-plugin/plugin.json      → Cursor plugin metadata
openclaw.plugin.json            → OpenClaw native plugin manifest
gemini-extension.json           → Gemini CLI extension metadata
(No new manifests needed — GitHub Copilot, Windsurf, Cline, JetBrains Junie, and Continue.dev read standard SKILL.md natively)
skills/ru-text/SKILL.md         → always-on typography + routing table (≤650 words, gated, cross-platform)
skills/ru-text/references/      → 9 domain files + addenda + sources (loaded on demand)
skills/ru-text/agents/openai.yaml → Codex skill metadata (Claude ignores)
skills/ru-text/agents/gemini.yaml → Gemini skill metadata (Claude ignores)
skills/ru-check/SKILL.md        → /ru-text:ru-check full-corpus check (fork context, read-only; moved from commands/ in a2d26d1)
skills/ru-score/SKILL.md        → /ru-text:ru-score 0–10 quality score (fork context, read-only; moved from commands/ in a2d26d1)
notion/ru-text-notion-skill.md  → Notion AI Custom Skill template (self-contained, ~1450 words)
notion/README.md                → Bilingual Notion setup guide (AI Skill + MCP workflow)
README.md + README.en.md        → bilingual docs (RU primary = GitHub default; EN in README.en.md; welcoming EN switcher atop README.md)
PRIVACY_POLICY.md               → zero data collection statement
assets/icon.png                 → 512×512 marketplace icon (Codex composerIcon; derived from logo-round.png)
```

Progressive disclosure: Claude reads SKILL.md first (typography rules, top stop-words, routing table), then loads domain-specific references only when the task requires them.

## Commands

```bash
claude plugins validate /path/to/ru-text    # validate manifest
claude plugins marketplace update ru-text   # refresh marketplace cache
/ru-text                                    # activate skill manually
/ru-text:ru-check <text>                    # comprehensive quality check (fork context)
/ru-text:ru-score <text>                    # text quality score 0–10 (fork context)
```

## Conventions

### Legal (apply to ALL text in repo: README, CHANGELOG, commits, comments)
- NEVER say "distilled from" — use "informed by" or "independently formulated"
- NEVER imply source authors endorse this plugin
- NEVER use author names in section headers as if they endorse (e.g., "Nora Gal's principles" → "Clean language principles, cf. N. Gal")
- IP notice references: Article 1259(5) of the Russian Civil Code, 17 USC §102(b), Berne Convention

### What belongs in this repository, and what does not

**Paid is whatever deterministically FIXES arbitrary user text. Public is the corpus — rules
in prose — and detectors whose input set is hard-wired into this repository.**

Adopted 16.08.2026, after two adversarial panels and an arbiter were asked to find the leak
and found none: no file here has ever transformed text, and the history says so —
`git log --all --diff-filter=A --pretty=format: --name-only -- '*.ts' '*.js' '*.mjs'` is
empty. The scripts under `tools/` are discipline gates — the count is deliberately not
stated here, because a hand-maintained number goes stale in the commit that adds a file, and
this one did. `check-typography.sh` hard-wires three filenames in `FILES` and takes no
arguments at all, and its own header records that the
same patterns over the corpus files report 53 «violations» that are all correct content.
`check-dogfood.sh` does not detect stop-words in prose — it counts them in the reference and
verifies the documentation quotes the right number.

Three more obvious phrasings were tried and each broke on a real file. «Prose vs code» would
exile `check-typography.sh`, the very gate that enforces this repository's own typography
claim. «Needed by the skill vs needed by the service» would exile `check-frozen.sh`, which
exists *for* the paid service yet guards public bytes.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [talkstream/ru-text](https://github.com/talkstream/ru-text) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->

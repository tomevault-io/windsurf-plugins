---
trigger: always_on
description: > This is an "idea file" in the spirit of [Karpathy's LLM Wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f). Paste it into any LLM agent (OpenAI Codex, Claude Code, OpenCode, Gemini Code Assist, or similar) and it will build and manage a wiki for you. The agent customizes the specifics; this file communicates the system.
---

# LLM Wiki — Agent Instructions

> This is an "idea file" in the spirit of [Karpathy's LLM Wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f). Paste it into any LLM agent (OpenAI Codex, Claude Code, OpenCode, Gemini Code Assist, or similar) and it will build and manage a wiki for you. The agent customizes the specifics; this file communicates the system.

> **Editing the llm-wiki repo itself?** This file is the portable wiki *protocol* for end users. The dev contract for the plugin codebase (testing, sync workflow, project structure, symlink invariant) lives in [`CLAUDE.md`](CLAUDE.md) — read it in addition to this file. Both Claude Code and Codex agents working on the repo should treat `CLAUDE.md` as the source of truth for repo-level workflow.

## What This Is

An LLM-compiled knowledge base. You (the LLM agent) are both the compiler and the query engine. Raw source documents are ingested, then incrementally compiled into a wiki of interconnected markdown articles. The human rarely edits the wiki directly — that's your domain.

**The metaphor**: raw sources are source code, you are the compiler, the wiki is the executable.

## Architecture

### Hub Path

Defaults to `~/wiki/`. Optionally configured via `~/.config/llm-wiki/config.json`:

```json
{ "hub_path": "~/Library/Mobile Documents/com~apple~CloudDocs/wiki" }
```

If the config exists, use `hub_path` (expand `~`) instead of `~/wiki/` everywhere below. Store as **HUB**. Use `/wiki config hub-path <path>` to set it up, or create the JSON manually.

### Hub (~/wiki/)

The hub is lightweight — NO content, just a registry.

```
~/wiki/                         # or custom path from config.json
├── wikis.json          # Registry of all topic wikis
├── _index.md           # Lists topic wikis with stats
├── log.md              # Global activity log
└── topics/
    ├── nutrition/      # Each topic is a full, isolated wiki
    ├── robotics/
    └── ...
```

### Topic Wiki (~/wiki/topics/<name>/)

All content lives here. One topic per wiki. Isolated indexes, focused queries.

```
~/wiki/topics/<name>/
├── .obsidian/                     # Obsidian vault config (optional)
├── _index.md                      # Master index: stats, navigation, recent changes
├── config.md                      # Title, scope, conventions
├── log.md                         # Activity log for this topic
├── inbox/                         # Drop zone — user dumps files here
│   └── .processed/
├── raw/                           # Immutable source material
│   ├── _index.md
│   ├── articles/*.md
│   ├── papers/*.md
│   ├── repos/*.md
│   ├── notes/*.md
│   └── data/*.md
├── wiki/                          # Compiled articles (LLM-maintained)
│   ├── _index.md
│   ├── concepts/*.md              # Bounded ideas
│   ├── topics/*.md                # Broad themes
│   ├── references/*.md            # Curated collections
│   └── theses/*.md                # Thesis investigations with verdicts
└── output/                        # Generated artifacts
    ├── _index.md
    └── *.md
```

### Local Wiki (.wiki/)

Same structure as a topic wiki but at `<project>/.wiki/`. Add `.wiki/` to `.gitignore`.

## Core Principles

1. **One topic, one wiki.** Never mix unrelated topics. The hub is just a registry.
2. **Indexes are navigation.** Every directory has `_index.md` with a contents table. Read indexes first, never scan blindly. Keep them current.
3. **Raw is immutable.** Once ingested, sources are never modified. All synthesis happens in `wiki/`.
4. **Articles are synthesized, not copied.** Draw from multiple sources, contextualize, connect. Think textbook, not clipboard.
5. **Dual-linking.** Every cross-reference uses both formats on the same line: `[[slug|Name]] ([Name](../category/slug.md))`. Obsidian reads the wikilink, the agent reads the markdown link, GitHub renders it. Not locked into any tool.
6. **Incremental by default.** Only compile new sources unless explicitly asked for full recompile.
7. **Honest gaps.** If the wiki doesn't have the answer, say so. Suggest what to ingest.
8. **Multi-wiki peek.** When querying, answer from the target wiki, then peek at sibling wiki `_index.md` files for overlap.
9. **Confidence scoring.** Articles get `confidence: high|medium|low` in frontmatter based on source quality.
10. **Activity log.** Append every operation to `log.md`. Format: `## [YYYY-MM-DD] operation | Description`. Never edit existing entries.

## File Formats

### _index.md (every directory)

```markdown
# [Directory Name] Index

> [One-line description]

Last updated: YYYY-MM-DD

## Contents

| File | Summary | Tags | Updated |
|------|---------|------|---------|
| [file.md](file.md) | One-sentence summary | tag1, tag2 | YYYY-MM-DD |

## Categories

- **category**: file1.md, file2.md

## Recent Changes

- YYYY-MM-DD: Description
```

Master `_index.md` additionally has Statistics and Quick Navigation sections.

### Raw Source (raw/)

```yaml
---
title: "Title"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nvk/llm-wiki](https://github.com/nvk/llm-wiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->

---
trigger: always_on
description: <!-- Knowledge priming document for AI agents and new contributors. -->
---

# AGENTS.md

<!-- Knowledge priming document for AI agents and new contributors. -->
<!-- Follows the Fowler Knowledge Priming structure (7 sections). -->
<!-- Keep this file to 1–3 pages. Contributor conventions are in this same file below. -->
<!-- Last updated: 2026-02-25 -->

---

## Architecture Overview

This repository is a **collection of reusable agent skills** following the Agent Skills open standard. Each skill is a self-contained directory containing a `SKILL.md` file (with YAML frontmatter) and optional executable scripts, reference docs, and static assets. Skills are portable: they work with Claude Code, Cursor, Codex CLI, OpenCode, Gemini CLI, and Amp — any agent that supports the standard.

The repo has no runtime server, build system, or package manager. All logic lives in Markdown instructions and Bash/Go scripts. Agents read `SKILL.md` upfront and load `references/` on demand, keeping token usage proportional to need. An `agentic-docs/` plugin provides a full documentation suite, while `shared/prompts/` holds common safety instructions injected across skills.

---

## Tech Stack and Versions

| Layer | Technology | Notes |
|-------|-----------|-------|
| Primary language | Bash | Hook scripts, setup scripts, skill orchestration |
| Secondary language | Go | HTML-to-Markdown binary in `markdown_crawl/scripts/` |
| Skill format | Markdown + YAML frontmatter | Parsed by agent runtimes |
| Config | JSON | `.claude-plugin/marketplace.json`, metrics files |
| Agent platforms | Claude Code, Cursor, Codex, OpenCode, Gemini CLI, Amp | Consuming runtimes, not dependencies |

No package.json, pyproject.toml, or go.mod at the repo root. The only `go.mod` is inside `skills/markdown_crawl/scripts/` for the Go binary.

---

## Curated Knowledge Sources

- [Agent Skills standard](https://agentskills.io) — official spec for SKILL.md format, frontmatter fields, and skill directories
- [Cursor Skills docs](https://cursor.com/docs/context/skills) — how Cursor loads and triggers skills
- [Claude Code hooks docs](https://code.claude.com/docs/en/hooks) — lifecycle events used by `setup-agent-cli-hooks`
- [Cloudflare Markdown for Agents](https://blog.cloudflare.com/markdown-for-agents/) — background for `markdown_crawl` skill
- [Cloudflare Markdown for Agents developer docs](https://developers.cloudflare.com/fundamentals/reference/markdown-for-agents/) — HTTP header spec (`Accept: text/markdown`, `X-Markdown-Tokens`)
- [Fowler — Knowledge Priming](https://martinfowler.com/articles/reduce-friction-ai/knowledge-priming.html) — rationale behind AGENTS.md structure
- `AGENTS.md` (this file) — contributor conventions (skill naming, frontmatter rules, review checklist)

---

## Project Structure

```
agent-skills/
├── skills/                        # One directory per skill
│   ├── _template/                 # Starter template — copy this to create a new skill
│   │   └── SKILL.md
│   ├── make-plan/                 # Instruction-only skill (no scripts)
│   │   └── SKILL.md
│   ├── markdown_crawl/            # Script-backed skill (Bash + Go binary)
│   │   ├── SKILL.md
│   │   ├── metrics.json           # Persistent usage metrics (auto-updated at runtime)
│   │   └── scripts/
│   │       ├── markdown_crawl.sh  # Main fetch/compare/stats script
│   │       ├── html_to_markdown   # Pre-built Go binary (Linux AMD64)
│   │       └── build.sh           # Rebuilds the Go binary
│   ├── prepare-agents-md/         # Instruction-only skill with reference docs
│   │   ├── SKILL.md
│   │   └── references/
│   │       └── fowler-structure.md
│   └── setup-agent-cli-hooks/     # Script-backed skill
│       ├── SKILL.md
│       └── scripts/
│           └── setup-agent-hooks.sh
├── agentic-docs/                  # Full documentation suite plugin (separate from skills)
│   ├── agents/doc-setup.md        # Agent instructions for generating docs
│   └── commands/                  # (placeholder)
├── shared/
│   └── prompts/common.md          # Shared safety prompt injected across skills
├── .claude-plugin/
│   └── marketplace.json           # Plugin marketplace metadata
├── AGENTS.md                      # This file
├── CLAUDE.md                      # Claude Code project settings
└── README.md                      # Human-readable project overview
```

---

## Naming Conventions

| Element | Convention | Example |
|---------|-----------|---------|
| Skill directory names | lowercase, hyphens | `make-plan`, `prepare-agents-md` |
| `name` field in SKILL.md frontmatter | Must match folder name exactly | `name: prepare-agents-md` |
| Script files | kebab-case `.sh` or descriptive binary name | `setup-agent-hooks.sh`, `html_to_markdown` |
| Reference files | `SCREAMING_SNAKE.md` or descriptive kebab | `fowler-structure.md` |
| Metrics/data files | kebab-case `.json` | `metrics.json` |
| YAML frontmatter tags | lowercase strings | `["hooks", "cli", "setup"]` |

**Note**: AGENTS.md documents `_template` uses underscores, but all real skills use hyphens. Follow hyphens for new skills.

---

## Code Examples

### Minimal SKILL.md (instruction-only skill)

```yaml
# from skills/make-plan/SKILL.md
---
name: make-plan
description: Make a detailed plan to fix or resolve an issue/problem. Follows a 6-step

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ducva) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->

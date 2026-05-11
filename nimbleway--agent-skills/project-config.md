---
trigger: always_on
description: **Nimble Web Search Skills** — agent skills that give any AI agent the ability to search, scrape, and extract structured data from any website using the Nimble CLI. Built following the [Agent Skills specification](https://agentskills.io/specification.md), compatible with Claude Code, Codex, Cursor, and any agent platform that supports the spec.
---

# CLAUDE.md

## What this repo is

**Nimble Web Search Skills** — agent skills that give any AI agent the ability to search, scrape, and extract structured data from any website using the Nimble CLI. Built following the [Agent Skills specification](https://agentskills.io/specification.md), compatible with Claude Code, Codex, Cursor, and any agent platform that supports the spec.

Two layers of skills:
- **Core data skills** (`skills/web-search-tools/`) — the raw capabilities: fetch a URL, run a search, build a reusable extraction agent
- **Business intelligence skills** (all other verticals) — one-command workflows that turn live web data into actionable reports

See `.claude-plugin/marketplace.json` for the full list of published skills.

Business skills are built on top of core skills — they call `nimble search` / `nimble extract` under the hood. The two core skills also form a feedback loop: web-expert runs agents built by agent-builder, and when a one-off lookup becomes recurring, agent-builder turns it into a reusable pipeline.

## Prerequisites

```bash
npm i -g @nimble-way/nimble-cli
export NIMBLE_API_KEY="your-key"   # or set in ~/.claude/settings.json under env
```

## Repo structure

```
skills/
  {vertical}/                    # Skills grouped by vertical
                                 #   business-research/, healthcare/, marketing/,
                                 #   productivity/, web-search-tools/
    {skill-name}/                #   Each skill = SKILL.md + optional references/
      SKILL.md                   #   Skill definition (frontmatter + instructions)
      references/                #   On-demand docs, loaded when needed
agents/                          # Shared sub-agent definitions (.md with frontmatter)
_shared/                         # Canonical shared references (synced into skills)
.claude-plugin/plugin.json       # Claude Code plugin manifest
.cursor-plugin/plugin.json       # Cursor plugin manifest
commands/                        # Slash commands
scripts/                         # Repo tooling
```

Verticals are just grouping folders — add new ones freely. `.claude-plugin/plugin.json` lists vertical directories explicitly; `.cursor-plugin/plugin.json` points to `./skills/` (all verticals). Update the relevant manifest when adding or removing verticals or agents.

## Commands

```bash
# Sync _shared/ references into business skill references/ folders
bash scripts/sync-shared.sh

# Test a skill locally — trigger it by name in a Claude Code session
claude "run competitor-intel for acme.com"
```

## Skill authoring

Every skill follows the [Agent Skills specification](https://agentskills.io/specification.md). Key rules for this repo:

### Writing style
- Clarity over cleverness. Specific over vague. Active voice over passive.
- Short paragraphs (2-4 sentences). One idea per section. Exception: intro taglines (one sentence after `# Skill Name`) are intentionally short.
- Challenge every token: "Does the agent really need this to do the job?"
- Say nothing notable rather than padding with fluff.

### Naming & structure
- Name: `{domain}-{action}`, lowercase, hyphenated. Folder name must match frontmatter `name`.
- Aim to keep SKILL.md under ~500 lines. Use progressive disclosure: frontmatter (always loaded) → body (on trigger) → `references/` directory (on demand). The `references/` directory IS the dedicated deeper layer — SKILL.md does not need a `## References` heading.

### SKILL.md frontmatter
```yaml
---
name: skill-name
description: |
  [What it does] + [When to use it] + [Key capabilities]. Max 1024 chars.
  Third-person voice. Include trigger phrases and negative triggers (use "Do NOT use for X — use Y instead" format).
allowed-tools:
  - Bash(nimble:*)
  - Bash(date:*)
metadata:
  author: Nimbleway
  version: 1.0.0
---
```

### DRY
- Shared patterns live in `_shared/` — edit there, then `bash scripts/sync-shared.sh`. The sync script copies `_shared/` files into each skill's `references/` directory — these synced copies are expected and not duplication.
- Never manually copy-paste shared logic into a SKILL.md — reference it via `references/`.
- Skill-specific logic (output format, entity research, agent team composition) stays in SKILL.md.
- When referencing shared patterns from SKILL.md, say "do X" and point to the playbook for "how X works" — don't restate the pattern inline.
- The restatement test: if `_shared/nimble-playbook.md` changed, would SKILL.md become
  wrong? If yes, SKILL.md is restating, not referencing. Grep for shared pattern
  signatures (`nimble map`, `nimble extract --`, `--render`, scaling tier tables) — if
  found inline in SKILL.md, it's a DRY violation.
- If a skill has multiple execution paths (e.g., geographic vs SaaS), each path must be first-class with its own discovery, scoring, output template, and error handling.

### Data access
- Use `nimble search` / `nimble extract` via Bash for web data access.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Nimbleway/agent-skills](https://github.com/Nimbleway/agent-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->

---
trigger: always_on
description: This file tells an AI agent (Claude Code, Codex, Cursor, Aider, Gemini CLI, or any other harness) what this repository is and how to install its skills.
---

# AGENTS.md — for arbitrary AI agents landing in this repo

This file tells an AI agent (Claude Code, Codex, Cursor, Aider, Gemini CLI, or any other harness) what this repository is and how to install its skills.

## What this repo is

A collection of **Anthropic-format skills** for academic research and writing, scoped to the needs of computer science / mathematics students and professors. The skills cover the full research-to-publication pipeline:

- `pull` — fetch external documents (webpages, PDFs, papers) into a local `sources/` folder as Markdown with provenance frontmatter.
- `ingest` — compile pulled sources into a cross-linked wiki at `wiki/`.
- `distill` — extract atomic claims (Zettelkasten-style) from sources into `notes/atomic/`.
- `sota` — produce a structured state-of-the-art review organized by user-relevant dimensions (paradigms, techniques, evaluation methods, domains), output to `sota/`.
- `review` — forensic audit of an academic document (thesis or paper); produces a narrative report (5–18 pages by level) plus a structured forensic assembly sibling.

All five skills ship in this repo under `skills/`:

```
skills/
├── pull/          (source-pulling pipeline via markitdown)
├── ingest/        (sources → cross-linked wiki)
├── distill/       (sources → atomic Zettel notes)
├── sota/          (corpus → structured state-of-the-art review)
└── review/        (academic document → narrative audit + forensic assembly)
```

Each is self-contained, ported with vault-specific paths removed and defaults set to the current working directory.

## How to install the skills (the action items for an agent)

The natural unit of action when an agent reads this repo is: **copy each subdirectory of `skills/` into the agent's skills directory**. Then the agent can invoke them by name (`/sota`, `/review`, etc.).

### Standard paths

| Agent | Skills directory |
|---|---|
| Claude Code (user-global) | `~/.claude/skills/` |
| Claude Code (project-local) | `<project>/.claude/skills/` |
| Codex | `~/.codex/skills/` *(check Codex docs for latest convention)* |
| Cursor | inspect Cursor's settings for skill / rule directory |
| Gemini CLI | depends on harness configuration |
| Generic | wherever the harness expects skill packages |

### The fast path: `install.sh`

If the harness has Bash and the agent can run shell commands, the cleanest install is:

```bash
./install.sh
```

This copies every subdir of `skills/` into `${SKILLS_DEST:-$HOME/.claude/skills/}`. Override the destination via env var:

```bash
SKILLS_DEST=$HOME/.codex/skills ./install.sh        # user-global, Codex
SKILLS_DEST=./.claude/skills ./install.sh           # project-local, Claude Code
SKILLS_DEST=./mi-proyecto/skills ./install.sh       # arbitrary
```

By default `install.sh` skips skills that already exist at the destination (does not overwrite). Pass `--force` to re-install over existing copies.

### The manual path: `cp`

If shell execution is not available or the agent prefers to act file-by-file:

```bash
mkdir -p ~/.claude/skills/
cp -r skills/sota         ~/.claude/skills/
cp -r skills/review       ~/.claude/skills/
# ... etc.
```

Each skill subdir is self-contained: it has its own `SKILL.md` (entry point for the main agent), optionally a `tools/` subdir with vendored scripts, and subagent prompt files (`synthesizer.md`, `assembler.md`, etc.) as siblings.

### Verifying install

After install, restart the agent session (or use the harness's "reload skills" mechanism if available). The skill descriptions should appear in the available-skills list, and invocations like `/sota <topic>` should resolve.

## Skill anatomy (so the agent can verify integrity)

Every skill in this repo follows the same shape:

```
skills/<name>/
├── SKILL.md           ← REQUIRED. YAML frontmatter with `name:` + `description:`,
│                        followed by the main agent prompt body.
├── synthesizer.md     ← optional, for skills with parallel-subagent phases
├── assembler.md       ← optional, for skills with a final synthesis step
├── (other .md)        ← additional subagent prompts as needed
└── tools/             ← optional, for vendored deterministic scripts
    └── *.py / *.sh
```

The `SKILL.md` YAML frontmatter must include `name:` (matching the directory name) and `description:` (telling the harness when to invoke). Both are surfaced in the harness's available-skills list at session start.

`SKILL.md` body is in **English**. Output artifacts (reports, narratives, observations) are produced in the language of the corpus being processed or in the user's interaction language; the harness handles the multilingual routing.

## What the skills assume about the workspace

Default output convention (relative to the agent's current working directory):

```
<cwd>/sources/        ← raw external documents (pull writes here)
<cwd>/wiki/           ← cross-linked synthesis (ingest writes here)
<cwd>/notes/atomic/   ← atomic-claim notes (distill writes here)
<cwd>/sota/           ← state-of-the-art reports (sota writes here)
<cwd>/reviews/        ← document audits (review writes here)
```

Every skill resolves its corpus/output paths in this order:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [matcom/ai-skills](https://github.com/matcom/ai-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->

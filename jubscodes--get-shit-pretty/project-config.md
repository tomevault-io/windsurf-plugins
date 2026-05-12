---
trigger: always_on
description: Design engineering system for Claude Code, OpenCode, Gemini, and Codex.
---

# GSP — Get Shit Pretty

Design engineering system for Claude Code, OpenCode, Gemini, and Codex.

## Git workflow

`main` is protected — all changes require a PR (no direct push). Use feature/release branches and squash merge via `gh pr merge --squash`.

## This repository

This repo is **both**:

- **Source and npm package** — where the GSP agentic framework is built, versioned, and published via `npm publish`. The `package.json` `files` field controls what ships: `.mcp.json`, `bin`, `scripts`, `gsp`.
- **GSP consumer** — GSP is installed here too (e.g. `.claude/` symlinks). You can run GSP workflows in this workspace while developing the framework.

Edit source under `gsp/`; the installer keeps runtimes in sync. Never edit inside `.claude/` (or other runtime dirs) directly — they point at or are populated from source.

## Architecture

Dual-diamond: **Branding** (discover → strategy → identity → patterns) + **Project** (brief → research → design → critique → build → review). Verbal identity is merged into brand-strategy (4 phases, not 5).

### Two skill layers

**Expertise skills** (knowledge owners): `gsp-color`, `gsp-typography`, `gsp-visuals`, `gsp-accessibility`, `gsp-style`

Own domain knowledge as sibling files (`domains/`, `references/`). Serve the full pipeline, not just one phase. Two consumption patterns:
- **Read** (passive) — pipeline skill or agent reads expertise skill's sibling files for domain context
- **Invoke** (active) — pipeline skill calls the expertise skill to run its logic (e.g. `--enrich`, `--validate`)

Rule: **never duplicate domain knowledge in pipeline skills.** If an expertise skill owns it, pipeline skills read or invoke.

**Pipeline skills** (orchestrators): `brand-research`, `brand-strategy`, `brand-identity`, `brand-guidelines`, `project-brief`, `project-research`, `project-design`, `project-critique`, `project-build`, `project-review`

Own workflow: state management, phase gates, agent spawning, user interaction. Consume domain knowledge from expertise skills. Produce artifacts to `.design/`.

### Skill architecture

Skills are lean routers. SKILL.md handles mode/flag parsing, context resolution, and delegation. Domain knowledge, questioning frameworks, output templates, and technical specs live in sibling files that the skill reads on demand.

Expertise skills use a `domains/` + `references/` structure:
```
gsp-color/
├── SKILL.md              ← thin router (~60-80 lines)
├── domains/
│   ├── palette.md        ← OKLCH generation spec
│   └── system.md         ← full color system direction
└── references/
    └── color-composition.md
```

The filesystem is the integration layer — skills produce artifacts to `.design/`; agents consume them. No skill-to-skill invocation except explicit `--enrich`/`--validate` calls.

## Pack structure

| Directory | Contents |
|-----------|----------|
| `gsp/skills/` | 34 skills — each is a `gsp-<name>/SKILL.md` directory with optional `domains/` and `references/` siblings |
| `gsp/agents/` | 12 subagents (`gsp-{name}.md`) |
| `gsp/hooks/` | Hooks (`hooks.json`) |
| `gsp/prompts/` | Reserved (agent methodology lives in skill `methodology/` directories) |
| `gsp/templates/` | Project/brand config, state, brief, roadmap templates |
| `.mcp.json` | Bundled MCP servers (GitHub, Figma) |
| `scripts/` | Hook scripts and utilities (at repo root) |

### Skill naming

Source skill directories under `gsp/skills/` use the `gsp-` prefix: `gsp-pretty/`, `gsp-brand-strategy/`, `gsp-style/`, etc. The one exception is `get-shit-pretty/` (entry point skill, `user-invocable: false`).

The `gsp-` prefix is part of the source directory name. The installer copies as-is — no renaming needed.

| Layer | Example |
|-------|---------|
| Source (`gsp/skills/`) | `gsp-style/SKILL.md` |
| Claude Code (`.claude/skills/`) | `gsp-style/` → `/gsp-style` |
| OpenCode (`.opencode/skills/`) | `gsp-style/` → `/gsp-style` |
| Gemini (`.gemini/skills/`) | `gsp-style/` → `/gsp-style` |
| Codex (`.agents/skills/`) | `gsp-style/` → `$gsp-style` |
| Vercel skills.sh | `gsp-style/` → `/gsp-style` |

Cross-references between skills use `gsp-` prefixed paths: `${CLAUDE_SKILL_DIR}/../gsp-style/styles/INDEX.yml`.

## Multi-runtime installer

`bin/install.js` converts Claude Code's native format into each runtime's expected format:

| Runtime | Skills location | Agents | Bundle location |
|---------|-----------------|--------|-----------------|
| Claude Code | `.claude/skills/` | `.claude/agents/` (12) | `.claude/{prompts,templates}/` |
| OpenCode | `.opencode/skills/` | `.opencode/agents/` (12) | `.opencode/{prompts,templates}/` |
| Gemini CLI | `.gemini/skills/` | `.gemini/agents/` (11, experimental) | `.gemini/{prompts,templates}/` |
| Codex CLI | **`.agents/skills/`** (not `.codex/`) | **None** (not supported) | `.codex/{prompts,templates}/` |

Skills are the single source for all runtimes — commands have been removed.

Key points:
- Codex has a **split layout**: config/bundles at `~/.codex/`, skills at `~/.agents/skills/`
- Codex does **not** install agents — agent `.md` files are skipped
- Tool names are mapped per runtime (e.g. `Bash` → `shell` for Codex, `run_shell_command` for Gemini)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jubscodes/get-shit-pretty](https://github.com/jubscodes/get-shit-pretty) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->

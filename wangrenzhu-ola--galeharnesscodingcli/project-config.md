---
trigger: always_on
description: This repository is **GaleHarnessCLI** — a complete coding-agent workflow system combining gh: prefix skills with HKTMemory vector knowledge base.
---

# Agent Instructions

This repository is **GaleHarnessCLI** — a complete coding-agent workflow system combining gh: prefix skills with HKTMemory vector knowledge base.

It contains:
- `plugins/galeharness-cli/` — GaleHarnessCLI workflow skills (gh: prefix) and agents
- `vendor/hkt-memory/` — HKTMemory v5.0 vector knowledge base, vendored in full
- the Bun/TypeScript CLI that converts plugins into other agent platform formats
- additional plugins under `plugins/`, such as `coding-tutor`
- `scripts/` — release tooling

`AGENTS.md` is the canonical repo instruction file. Root `CLAUDE.md` exists only as a compatibility shim for tools and conversions that still look for it.

## Quick Start

```bash
bun install
bun test                  # full test suite
bun run release:validate  # check plugin/marketplace consistency
```

## GaleHarnessCLI Conventions

### HKTMemory

`vendor/hkt-memory/` contains HKTMemory v5.0. First-time setup:
```bash
bash vendor/hkt-memory/install.sh
```

Required environment variables:
```
HKT_MEMORY_API_KEY=<your key>
HKT_MEMORY_BASE_URL=https://open.bigmodel.cn/api/paas/v4/
HKT_MEMORY_MODEL=embedding-3
```

Quick test:
```bash
hkt-memory stats
```

### SKILL Naming

All workflow skills use `gh:` prefix:
- `gh:compound` — document a solved problem (stores to HKTMemory automatically)
- `gh:plan`, `gh:work`, `gh:review`, `gh:debug`, etc.

The `learnings-researcher` agent queries HKTMemory before grep-searching `docs/solutions/`.

## Working Agreement

- **Branching:** Create a feature branch for any non-trivial change. If already on the correct branch for the task, keep using it; do not create additional branches or worktrees unless explicitly requested.
- **Safety:** Do not delete or overwrite user data. Avoid destructive commands.
- **Testing:** Run `bun test` after changes that affect parsing, conversion, or output.
- **Release versioning:** Releases are prepared by release automation, not normal feature PRs. The repo now has multiple release components (`cli`, `galeharness-cli`, `coding-tutor`, `marketplace`). GitHub release PRs and GitHub Releases are the canonical release-notes surface for new releases; root `CHANGELOG.md` is only a pointer to that history. Use conventional titles such as `feat:` and `fix:` so release automation can classify change intent, but do not hand-bump release-owned versions or hand-author release notes in routine PRs.
- **Linked versions (cli + galeharness-cli):** The `linked-versions` release-please plugin keeps `cli` and `galeharness-cli` at the same version. This is intentional — it simplifies version tracking across the CLI and the plugin it ships.
- **Output Paths:** Keep OpenCode output at `opencode.json` and `.opencode/{agents,skills,plugins}`. For OpenCode, commands go to `~/.config/opencode/commands/<name>.md`; `opencode.json` is deep-merged (never overwritten wholesale).
- **Scratch Space:** Two options depending on what the files are for:
  - **Workflow state** (`.context/`): Files that other skills or agents in the same session may need to read — plans in progress, gate files, inter-skill handoff artifacts. Namespace under `.context/galeharness-cli/<workflow-or-skill-name>/`, add a per-run subdirectory when concurrent runs are plausible, and clean up after successful completion unless the user asked to inspect them or another agent still needs them.
  - **Throwaway artifacts** (`mktemp -d`): Files consumed once and discarded — captured screenshots, stitched GIFs, intermediate build outputs, recordings. Use OS temp (`mktemp -d -t <prefix>-XXXXXX`) so they live outside the repo tree entirely. No `.gitignore` needed, no risk of accidental commits, OS handles cleanup.
  - **Rule of thumb:** If another skill might read it, `.context/`. If it gets uploaded/consumed and thrown away, OS temp. Durable outputs like plans, specs, learnings, and docs belong in neither — they go in `docs/`.
- **Character encoding:**
  - **Identifiers** (file names, agent names, command names): ASCII only — converters and regex patterns depend on it.
  - **Markdown tables:** Use pipe-delimited (`| col | col |`), never box-drawing characters.
  - **Prose and skill content:** Unicode is fine (emoji, punctuation, etc.). Prefer ASCII arrows (`->`, `<-`) over Unicode arrows in code blocks and terminal examples.

## Directory Layout

```
src/              CLI entry point, parsers, converters, target writers
plugins/          Plugin workspaces (galeharness-cli, coding-tutor)
.claude-plugin/   Claude marketplace catalog metadata
tests/            Converter, writer, and CLI tests + fixtures
docs/             Solutions and target specs
```

## Repo Surfaces

Changes in this repo may affect one or more of these surfaces:

- `galeharness-cli` under `plugins/galeharness-cli/`
- the Claude marketplace catalog under `.claude-plugin/`
- the converter/install CLI in `src/` and `package.json`
- secondary plugins such as `plugins/coding-tutor/`

Do not assume a repo change is "just CLI" or "just plugin" without checking which surface owns the affected files.

## Plugin Maintenance

When changing `plugins/galeharness-cli/` content:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wangrenzhu-ola/GaleHarnessCodingCLI](https://github.com/wangrenzhu-ola/GaleHarnessCodingCLI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->

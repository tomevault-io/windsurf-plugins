---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository status

OMCR is at **v0.1**, work-in-progress. The only tagged releases are `v0.1.0` and `v0.1.1`; the plugin manifest (`.claude-plugin/plugin.json`) declares `0.1.0`. Everything currently on `main` — including the orchestration engines and the autonomous supervisor — is unreleased v0.1 work. There is no v0.2 / v0.3 / v0.4 — those labels showed up in earlier docs by mistake and are being removed. Current tree ships:
- 6 research-team agents (`agents/`)
- 10 slash commands (`commands/`) — 4 setup/workflow (`/omcr-setup`, `/start-research`, `/todofig`, `/sync`) + 6 orchestration engines (`/iterate-revision`, `/literature-sweep`, `/respond-reviewer`, `/figure-bake`, `/outline-expand`, `/supervisor-drive`)
- 14 skills (`skills/`) — 7 setup/workflow skills + 1 primitive (`orchestrate`) + 6 engine skills backing the orchestration commands
- 4 lightweight hooks (`hooks/`)
- a canonical memory schema (`templates/MEMORY.template.md`)
- canonical orchestration-state schemas (`develop/example-state/` — tracked reference for `.claude/omcr-state/{paper,reviews,citations,figures,rebuttals}.json` + `_run-log.jsonl`)
- the plugin manifest (`.claude-plugin/plugin.json`)
- one worked example preset (`examples/neuro-fmri/` — neuro-flavored analysis-implementer body + redacted MEMORY.md skeletons)
- wiki documentation (`wiki/`) — 15 markdown pages, mirrors to GitHub Wiki via `wiki/README.md` instructions

MIT licensed (`LICENSE`, 2026 Young-Eun Lee). No build chain, no npm — plain markdown plus shell scripts loaded directly by Claude Code.

## Project intent

Public release on GitHub as the **research companion** to the upstream `oh-my-claudecode` project. The "research" framing means curated agents and conventions tailored to producing papers — not a runnable application. Treat additions through that lens; if a new asset is general-purpose orchestration, it likely belongs in upstream OMC instead.

## Conventions baked into the repo

- `.gitignore` excludes `.claude/` and `.omc/` — per-user / OMC plugin state stays local and must not be committed. If you create shared Claude Code assets (slash commands, agents, hooks) intended to ship with the repo, they need a different location or a targeted un-ignore, not a blanket removal of the rule.
- `.env` is gitignored — secrets stay out of the tree.

## External reference symlinks

The repo root holds symlinks to other local checkouts the maintainer wants accessible while working here:

- [`oh-my-claudecode/`](oh-my-claudecode/) → maintainer's checkout of the upstream `oh-my-claudecode` project. Source material this "research" repo is built around — read it for prior art, naming, structure, and "how does upstream do X" (`oh-my-claudecode/README.md`, `CLAUDE.md`, `AGENTS.md`).
- [`DoD-Agent/`](DoD-Agent/) → maintainer's checkout of a separate research project (`DoD-Agent`). Available as cross-reference; its own `CLAUDE.md` describes its scope. Don't assume content from `DoD-Agent` belongs in this repo unless the user says so.

Rules that apply to **all** symlinks in this section:

- Targets are **absolute paths on the maintainer's machine**, so each entry is gitignored individually and will not exist for other contributors or CI. Do not commit them, do not rewrite paths to make them portable, and do not assume they resolve anywhere except the maintainer's machine.
- Treat contents as **read-only reference**. Never edit files through these symlinks — writes would land in the external working tree, not this repo. If something from one of them should live here, copy the relevant excerpt into a tracked file under this repo.
- If a symlink is missing, ask the user to recreate it with `ln -s <absolute-path> <name>` rather than guessing at the external structure.
- When adding a new external symlink, follow this same pattern: create at repo root, add `/<name>` to `.gitignore`, list it in this section with a one-line purpose.

## File layout

```
oh-my-claudecode-research/
├── .claude-plugin/plugin.json        # plugin manifest (registers agents/commands/skills/hooks)
├── .gitattributes                    # LF line-ending normalization
├── agents/                           # 6 generic core agents (kebab-case, frontmatter required)
│   ├── supervisor.md
│   ├── analysis-implementer.md
│   ├── paper-writer.md
│   ├── figure-descriptor.md
│   ├── reviewer.md
│   └── literature-curator.md         # bibliography curator + BibTeX/summary-table owner
├── commands/                         # 10 thin dispatcher slash commands — all delegate to a matching skill
│   ├── omcr-setup.md                 # /omcr-setup → skills/omcr-setup/
│   ├── start-research.md             # /start-research → skills/start-research/
│   ├── todofig.md                    # /todofig → skills/todofig/
│   ├── sync.md                       # /sync → skills/sync/
│   ├── iterate-revision.md           # /iterate-revision → skills/iterate-revision/
│   ├── literature-sweep.md           # /literature-sweep → skills/literature-sweep/
│   ├── respond-reviewer.md           # /respond-reviewer → skills/respond-reviewer/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [youngeun1209/oh-my-claudecode-research](https://github.com/youngeun1209/oh-my-claudecode-research) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->

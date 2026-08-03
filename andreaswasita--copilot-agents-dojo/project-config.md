---
trigger: always_on
description: Authoritative reference for humans (and AI assistants) **modifying** the dojo itself. If you're just *using* the dojo in your own repo, start with [`README.md`](README.md). If you're inside an agent session running on the dojo, the runtime prompt is [`.github/copilot-instructions.md`](.github/copilot-instructions.md).
---

# Copilot Agents Dojo — Contributor Guide

Authoritative reference for humans (and AI assistants) **modifying** the dojo itself. If you're just *using* the dojo in your own repo, start with [`README.md`](README.md). If you're inside an agent session running on the dojo, the runtime prompt is [`.github/copilot-instructions.md`](.github/copilot-instructions.md).

This file is load-bearing. Reviewers may reject PRs that violate the rules below.

---

## Project Structure

File counts shift constantly — don't treat the tree below as exhaustive. The canonical source is the filesystem.

```
copilot-agents-dojo/
├── AGENTS.md                          # this file — contributor reference
├── README.md                          # user-facing onboarding
├── SOUL.md                            # agent identity charter (who / how / limits)
├── skills.md                          # GENERATED — skills index grouped by tier
├── spec/
│   └── copilot-skills-spec.md         # the HARDLINE skill spec (v1)
├── template/
│   └── SKILL.md                       # canonical starter for new skills
├── skills/                            # core + practical skills (always discoverable)
│   ├── plan-before-code/SKILL.md      # tier: core
│   ├── code-review/SKILL.md           # tier: practical
│   └── …
├── optional-skills/                   # heavy / niche skills (installed explicitly)
├── scripts/
│   ├── init.sh                        # scaffold tasks/{todo,lessons}.md
│   ├── verify.sh                      # the lint/test/invariant gate
│   ├── run-checks.ps1                 # Windows parity for verify.sh
│   ├── regen-skills-index.sh          # rebuilds skills.md from frontmatter
│   ├── lesson-updater.sh              # cache-aware skill amendments
│   └── curator.sh                     # skill lifecycle (pin/archive/restore)
├── tasks/
│   ├── todo.md                        # current plan (rollup of tasks/board/)
│   ├── lessons.md                     # postmortem log
│   └── board/                         # durable per-task markdown files
├── agents/                            # persona briefs (architect, TPM, etc.)
├── mcp/
│   ├── registry.yaml                  # MCP server catalog
│   ├── servers/                       # per-server JSON manifests
│   └── scripts/                       # mcp-subprocess wrappers
├── cli/
│   └── dojo_cli/                      # optional Python CLI (marketplace + scanner)
├── .github/
│   ├── copilot-instructions.md        # runtime prompt for sessions in this repo
│   ├── known-pitfalls.md              # imperative DO NOT register
│   └── workflows/dojo-enforce.yml     # PR enforcement
└── .dojo/                             # per-clone state (telemetry, profiles); gitignored
    └── skill-usage.json               # curator telemetry sidecar
```

---

## Adding a Skill

1. Copy `template/SKILL.md` to `skills/<name>/SKILL.md` (or `optional-skills/<name>/` for heavyweight skills).
2. Fill in **all required frontmatter** — see [`spec/copilot-skills-spec.md`](spec/copilot-skills-spec.md) §1.
3. Write the body in the **required section order** — spec §2.
4. Reference real Copilot tools in backticks (`view`, `edit`, `grep`, `glob`, `powershell`, `web_fetch`, `task`). NOT bare shell utilities — spec §3.
5. If the skill needs deterministic logic, add `scripts/` (ship `.sh` + `.ps1` for cross-platform) and `tests/`.
6. Run `scripts/verify.sh` locally. It must pass.
7. Open the PR. Reviewer checks against `.github/known-pitfalls.md` + the spec.

The full reviewer checklist lives in `.github/known-pitfalls.md`.

---

## Adding a CLI Command

The optional Python CLI lives in `cli/dojo_cli/`. Commands are centralized in `cli/dojo_cli/registry.py` (see Phase 5 of the roadmap) — adding a command is one entry in `COMMAND_REGISTRY`. `app.py`, `--help`, `marketplace.py`, and shell completion all derive from it.

Until that registry exists, follow the per-file pattern in `app.py` but keep new commands tiny and dependency-free — the CLI is a convenience, never a hard dependency.

---

## Testing

**Always use `scripts/verify.sh`** (or `scripts/run-checks.ps1` on Windows). The wrapper enforces hermetic env parity with CI:

| | Without wrapper | With wrapper |
|---|---|---|
| Credentials | Whatever is in your env | All `*_TOKEN` / `*_API_KEY` unset |
| Timezone | Local | UTC |
| Locale | Local | C.UTF-8 |
| `DOJO_ROOT` | Inherited | Temp dir per skill test |

Direct `pytest` calls on a developer machine diverge from CI in ways that have caused "works locally, fails in CI" incidents in other projects.

```bash
scripts/verify.sh                       # full gate
scripts/verify.sh tests                 # only the pytest suite
scripts/verify.sh spec                  # only the spec/frontmatter invariants
scripts/verify.sh --check               # CI mode: fail on any drift
```

### Test Discipline — No Change-Detector Tests


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [andreaswasita/copilot-agents-dojo](https://github.com/andreaswasita/copilot-agents-dojo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->

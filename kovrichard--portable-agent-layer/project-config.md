---
trigger: always_on
description: Working notes for any AI agent contributing to this repository.
---

# AGENTS.md

Working notes for any AI agent contributing to this repository.

> User-facing intro lives in `README.md`. This file is for agents — start here.

## What is PAL

The Portable Agent Layer is a cross-platform, cross-agent infrastructure for carrying personal AI context (TELOS, memory, skills, hooks) between machines and between AI runtimes (Claude Code, opencode, Cursor, Codex). It ships as a CLI (`pal`) plus a curated set of skills, hooks, and tooling that targets each agent's native config format.

The repo has two layers you'll spend most of your time in:

- **`src/`** — the CLI, the runtime hooks PAL itself installs (security validators, session-end handlers, etc.), and tools for managing memory, telos, threads, wisdom frames, opinion tracking, etc.
- **`assets/skills/<name>/`** — the skills PAL ships. Each skill is a self-contained folder with a `SKILL.md`, often a `tools/` subdir with TypeScript scripts, plus optional `template/`, `demo/`, and `theme-base/` subdirs (e.g. for the presentation skill).

When PAL is installed, `~/.pal/skills/<name>/` may be a directory junction back to `assets/skills/<name>/`. Edit at the repo source path, never the `~/.pal/...` path — the junction can mask whether you're touching tracked files.

## Repository layout

```
.
├── .agents/                  # this repo's dev hooks (not the PAL runtime hooks)
│   └── hooks/                # bun-run TypeScript hooks called by Stop events
├── .claude/                  # project-level Claude Code config (committed)
│   └── settings.json         # Stop hook + permission allowlist
├── .cursor/                  # project-level Cursor config (committed)
│   └── hooks.json            # stop event → same hook chain
├── .opencode/                # opencode plugin folder (committed)
│   └── plugins/lint.ts       # session.idle handler → same hook chain
├── .husky/                   # pre-commit / pre-push hooks (lint-staged + biome)
├── assets/
│   └── skills/               # all PAL-shipped skills
│       └── <name>/
│           ├── SKILL.md      # agent-facing skill spec (instructions YOU follow)
│           ├── README.md     # human-facing skill blurb
│           ├── tools/*.ts    # bun-run scripts the skill exposes
│           ├── template/     # scaffolding for new artefacts (optional)
│           ├── demo/         # showcase content (optional)
│           ├── theme-base/   # brand-neutral assets (presentation skill)
│           └── vendor/       # vendored 3rd-party libs (presentation skill)
├── src/
│   ├── cli/                  # `pal` CLI — entrypoint at src/cli/index.ts
│   │   ├── setup-identity.ts
│   │   └── setup-telos.ts
│   ├── hooks/                # PAL runtime hooks (installed into target agents)
│   │   └── lib/              # shared hook helpers; semi-static.ts is the single registry for context sources
│   ├── targets/              # per-agent install/uninstall logic
│   │   ├── claude/
│   │   ├── opencode/
│   │   ├── cursor/
│   │   ├── copilot/
│   │   └── lib.ts
│   └── tools/                # tools the agent invokes at runtime
│       ├── agent/            # synthesize, thread, wisdom-frame, analyze
│       ├── self-model.ts
│       ├── relationship-reflect.ts
│       └── token-cost.ts
├── test/                     # bun test files; one per concern
├── biome.json                # lint + format config
├── knip.json                 # dead-code/dep checker config
├── package.json
├── AGENTS.md                 # ← you are here
├── CLAUDE.md                 # symlink → AGENTS.md
└── README.md                 # user-facing intro
```

## Running and testing

This repo uses [Bun](https://bun.sh) ≥ 1.3.0 — never `npm`, `pnpm`, or `node`.

```bash
# install deps (frozen on CI)
bun install

# run the full test suite (currently ~26 files, ~304 tests)
bun test

# run a single test file
bun test test/cli.test.ts
bun test test/doctor.test.ts

# run one test by name pattern
bun test --test-name-pattern "scaffolds telos"
```

Check, typecheck, knip, test — each script and its agent-hook wrapper:

| Script                | What it does                            | Hook wrapper                   |
| --------------------- | --------------------------------------- | ------------------------------ |
| `bun run check`       | Biome lint + format (read-only)         | `.agents/hooks/check.ts`       |
| `bun run check-write` | Biome lint + format with `--write`      | (manual, not in stop chain)    |
| `bun run type-check`  | `tsc --noEmit`                          | `.agents/hooks/type-check.ts`  |
| `bun run knip`        | Knip dead-code / unused-deps scan       | `.agents/hooks/knip.ts`        |
| `bun run test`        | Bun test runner                         | (run by CI; not in stop chain) |

Run the CLI itself in dev mode against a sandboxed home:

```bash
PAL_HOME=./.test-home bun src/cli/index.ts cli help
PAL_HOME=./.test-home bun src/cli/index.ts cli init
```

`.test-home/` is wiped at the start of `bun test test/cli.test.ts`.

## The `.agents/hooks/` system

All three checks above are wired into the **Stop / session-end** event of every agent that has a config file in this repo. When an agent stops generating in this repo, it runs:

```
bun .agents/hooks/check.ts
bun .agents/hooks/type-check.ts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kovrichard/portable-agent-layer](https://github.com/kovrichard/portable-agent-layer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-12 -->

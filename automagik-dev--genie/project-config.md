---
trigger: always_on
description: bun run check        # Full gate: typecheck + lint + dead-code + test
---

# Genie CLI

## Commands

```bash
bun run check        # Full gate: typecheck + lint + dead-code + test
bun run build        # Bundle to dist/genie.js (bun target, minified, single file)
bun run typecheck    # tsc --noEmit
bun run lint         # biome check .
bun run dead-code    # bunx knip (has pre-existing false positives for biome/commitlint/husky)
bun test             # All tests
bun test src/lib/wish-state.test.ts  # Single file
```

## Docs

`docs/` is a symlink to `.docs-vendor/genie/` where `.docs-vendor` is a git submodule of `automagik-dev/docs` (Mintlify, public site at automagik.dev). Engineers see and edit `docs/` as if it were a regular subfolder of the genie repo — the submodule machinery is mostly invisible.

- **Operator-facing pages** (e.g., `docs/installation.mdx`, `docs/security/key-rotation.mdx`, `docs/incident-response/canisterworm.mdx`) appear on the public Mintlify site at `automagik.dev/genie/...`.
- **Engineering-internal pages** live under `docs/_internal/` (architecture deep-dives, observability internals, agent-frontmatter contracts, CLI reference dumps, spawn-flow runbooks, detector specs). These are excluded from the public Mintlify build via `**/_internal/` in `automagik-dev/docs/.mintignore` — visible inside the genie repo, hidden from public docs.

**Workflow when editing docs:**

```bash
# Make changes (the symlink follows into .docs-vendor/genie/)
$EDITOR docs/installation.mdx

# Commit + push the docs change to automagik-dev/docs
cd .docs-vendor
git checkout -b feat/<topic>
git add genie/installation.mdx
git commit -m "docs(genie): ..."
git push -u origin feat/<topic>
gh pr create --base main

# After the docs PR merges, bump the genie superproject pointer
cd ..   # back to genie repo root
git submodule update --remote .docs-vendor
git add .docs-vendor
git commit -m "chore: bump .docs-vendor to docs main"
```

CI in `automagik-dev/genie` runs `actions/checkout@v4` with `submodules: recursive` for any workflow that needs docs content (`docs-lint.yml`, `runbook-test.yml`); the rest of CI ignores the submodule.

## Architecture

```
src/genie.ts                    CLI entry point (commander)
src/lib/                        Core modules (state, registry, locking, messaging, providers)
src/lib/transcript.ts           Provider-agnostic transcript abstraction (Claude + Codex)
src/lib/codex-logs.ts           Codex JSONL parsing + SQLite discovery
src/lib/claude-logs.ts          Claude log parsing + transcript adapter
src/term-commands/              CLI command handlers
  agent/                        genie agent — spawn, stop, resume, kill, list, show, log, send, answer, register, directory, inbox, brief
  task/                         genie task — extends core CRUD with status, reset, board, project, releases, type
  team/                         genie team — create, hire, fire, list, disband
  exec/                         genie exec — list, show, terminate (debug)
src/hooks/                      Git hook system (branch-guard, auto-spawn, identity-inject)
src/genie-commands/             Setup/utility commands (setup, doctor, update, session)
src/types/                      Shared types (genie-config Zod schema)
skills/                         Skill prompt files (brainstorm, wish, work, review, etc.)
```

## CLI Namespaces

Top-level aliases (`genie spawn`, `genie kill`, etc.) are shortcuts for the `genie agent` namespace. Both forms work identically.

### Agent Commands
```bash
# Top-level aliases (shortcuts)
genie spawn <name>                    # Alias for: genie agent spawn <name>
genie kill <name>                     # Alias for: genie agent kill <name>
genie stop <name>                     # Alias for: genie agent stop <name>
genie resume [name]                   # Alias for: genie agent resume [name]
genie ls                              # Alias for: genie agent list
genie log [agent]                     # Alias for: genie agent log [agent]
genie read <name>                     # Read terminal output from agent pane
genie history <name>                  # Show compressed session history
genie answer <name> <choice>          # Alias for: genie agent answer <name> <choice>

# Full namespace commands
genie agent spawn <name>              # Spawn agent (resolves from directory or built-ins)
genie agent list                      # List agents with runtime status
genie agent log <name>                # Unified log (default)
genie agent log <name> --raw          # Pane capture
genie agent log <name> --transcript   # Compressed transcript
genie agent send '<msg>' --to <name>  # Direct message (hierarchy-enforced)
genie agent send '<msg>' --broadcast  # Team broadcast
genie agent inbox                     # View inbox
genie agent brief --team <name>       # Cold-start summary
genie agent answer <name> <choice>    # Answer prompt
genie agent show <name>               # Agent + executor detail
genie agent stop/kill/resume <name>   # Lifecycle management
genie agent register <name>           # Register agent locally + Omni
genie agent directory [name]          # List/show directory entries
```

### Task Commands
```bash
genie task create --title 'x'         # Create task
genie task list                       # List tasks

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [automagik-dev/genie](https://github.com/automagik-dev/genie) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->

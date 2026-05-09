---
trigger: always_on
description: Run `br prime` for workflow context. Use `bv --robot-triage` to find prioritized work.
---

# Agent Instructions

Run `br prime` for workflow context. Use `bv --robot-triage` to find prioritized work.

---

## Agent Warnings

### Do NOT Use `br edit`

**WARNING:** `br edit` opens an interactive editor (`$EDITOR`) which Claude Code cannot use. It will hang indefinitely.

Use `br update` with flags instead:
```bash
br update <id> --title "new title"
br update <id> --description "new description"
br update <id> --design "design notes"
br update <id> --notes "additional notes"
br update <id> --acceptance "acceptance criteria"
br update <id> --status in_progress
br update <id> --add-note "Session end: <context>"
```

### Non-Interactive Shell Commands

Some shell environments alias `cp`, `mv`, and `rm` to their `-i` (interactive) variants,
which causes commands to hang waiting for confirmation. **Use force flags specifically to
bypass these interactive alias prompts**, not as a blanket rule for all file operations:

```bash
cp -f source dest           # Avoids hanging if cp is aliased to cp -i
mv -f source dest           # Avoids hanging if mv is aliased to mv -i
rm -f file                  # Avoids hanging if rm is aliased to rm -i
rm -rf directory            # Avoids hanging if rm is aliased to rm -i
cp -rf source dest          # Avoids hanging if cp is aliased to cp -i
```

> **WARNING:** Force flags do NOT override DCG protection. DCG may still block destructive
> commands on protected paths (e.g., `rm -rf ./src`) even with `-f`. This is intentional.
> See `.claude/rules/destructive-command-guard.md` for allowlisted exceptions.

Other commands that may prompt:
- `scp` — use `-o BatchMode=yes`
- `ssh` — use `-o BatchMode=yes`
- `apt-get` — use `-y` flag
- `brew` — use `HOMEBREW_NO_AUTO_UPDATE=1`

### Use `bv --robot-*` for Triage and `br --json` for Mutations

For querying, triage, and analysis, use `bv` robot commands (output JSON by default). See [Using bv as an AI sidecar](#using-bv-as-an-ai-sidecar) for the full command reference, output format, filtering, and usage patterns.

For mutations and detailed single-issue views, use `br` with `--json`:
```bash
br show <id> --json
br list --json
br compact --analyze --json
```

---

## When to Use OpenSpec

Skip OpenSpec for work you can describe in a single bead description. Use OpenSpec when the work needs structured thinking — multiple capabilities, architectural decisions, or complex requirements.

| Situation | Action |
|-----------|--------|
| New feature/capability | `br create` epic, then `/opsx:ff` to plan |
| Need to think through a problem | `/opsx:explore` |
| Need structured planning | `/opsx:ff <name>` or `/opsx:new <name>` |

---

## Priority Scale

Beads uses numeric priorities 0-4 (or P0-P4). Do NOT use "high"/"medium"/"low".

| Priority | Meaning | Use When |
|----------|---------|----------|
| P0 | Critical | Production down, data loss, security breach |
| P1 | High | Blocks other work, must fix this session |
| P2 | Medium | Standard work, default for most tasks |
| P3 | Low | Nice to have, do when convenient |
| P4 | Backlog | Someday/maybe, future consideration |

```bash
br create "Fix auth bug" -t bug -p 0 -d "..."   # Critical
br create "Add feature" -t task -p 2 -d "..."    # Standard
```

---

## Dependencies

Use `br dep` to express blocking relationships between issues:

```bash
br dep add <issue> <depends-on>     # issue depends on depends-on
bv --robot-alerts                   # Show blocked issues and blocking cascades
br show <id>                        # See blocking/blocked-by for an issue
```

Example:
```bash
br create "Implement API" -t task -p 2 -d "..."       # → br-abc
br create "Write API tests" -t task -p 2 -d "..."     # → br-def
br dep add br-def br-abc            # Tests depend on API implementation
```

---

## Sync Mechanics

**Note:** `br` is non-invasive and never executes git commands. After `br sync --flush-only`, you must manually run `git add .beads/ && git commit`.

**Always run `br sync --flush-only` at the end of your session.** It exports pending changes to JSONL. Then commit and push manually:

```bash
br sync --flush-only
git add .beads/
git commit -m "sync beads"
git push
```

### Merge Conflicts in `.beads/issues.jsonl`

Hash-based IDs make conflicts rare, but if they occur:

```bash
# WARNING: This discards ALL local beads changes in favor of remote.
# Only use when you are certain the remote version is correct.
git checkout --theirs .beads/issues.jsonl   # Accept remote version
br import -i .beads/issues.jsonl            # Re-import to rebuild DB
```

Or for manual resolution: merge the file, then run `br import`.

### Test Database Isolation

**Never pollute the production database with test issues.** Use `BEADS_DB` for manual testing:

```bash
BEADS_DB=/tmp/test.db br create "Test issue" -p 1
```

---

### Using bv as an AI sidecar

bv is a graph-aware triage engine for Beads projects (.beads/beads.jsonl). Instead of parsing JSONL or hallucinating graph traversal, use robot flags for deterministic, dependency-aware outputs with precomputed metrics (PageRank, betweenness, critical path, cycles, HITS, eigenvector, k-core).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JordanChoo/claude-code-starter](https://github.com/JordanChoo/claude-code-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->

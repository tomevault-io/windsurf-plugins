---
trigger: always_on
description: This project uses **bd** (beads) for issue tracking. Run `bd prime` for full workflow context.
---

# Agent Instructions

This project uses **bd** (beads) for issue tracking. Run `bd prime` for full workflow context.

> **Architecture in one line:** Issues live in a Dolt database (`.beads/`).
> Bead data is replicated off-machine via Dolt's git-compatible wire protocol:
> the Dolt remote `origin` points at the same GitHub origin as the code, and
> issue history rides under `refs/dolt/data` there. After committing local `bd`
> writes, run `bd dolt push` to sync; `bd dolt pull` (or `bd bootstrap` on a
> fresh clone) hydrates from the remote. `.beads/issues.jsonl` is a passive
> export, not the wire protocol.

## Quick Reference

```bash
bd ready              # Find available work
bd show <id>          # View issue details
bd update <id> --claim  # Claim work atomically
bd close <id>         # Complete work
```

## Non-Interactive Shell Commands

**ALWAYS use non-interactive flags** with file operations to avoid hanging on confirmation prompts.

Shell commands like `cp`, `mv`, and `rm` may be aliased to include `-i` (interactive) mode on some systems, causing the agent to hang indefinitely waiting for y/n input.

**Use these forms instead:**
```bash
# Force overwrite without prompting
cp -f source dest           # NOT: cp source dest
mv -f source dest           # NOT: mv source dest
rm -f file                  # NOT: rm file

# For recursive operations
rm -rf directory            # NOT: rm -r directory
cp -rf source dest          # NOT: cp -r source dest
```

**Other commands that may prompt:**
- `scp` - use `-o BatchMode=yes` for non-interactive
- `ssh` - use `-o BatchMode=yes` to fail instead of prompting
- `apt-get` - use `-y` flag
- `brew` - use `HOMEBREW_NO_AUTO_UPDATE=1` env var

## Critical Don'ts (read before you do anything)

These are hard-won lessons. Each was learned the painful way. Run `bd memories` at session start for the full list of operational gotchas; the rules below are the ones serious enough to live here permanently.

- **Don't write local session summaries.** Never create `COMPLETION_SUMMARY_*.md`, `VERIFICATION_*.md`, `NOTES_*.md`, or similar files anywhere in the repo unless the user **explicitly asks for one**. The bead is the canonical record — capture verification, evidence, snapshots, contrast calculations, and reasoning via `bd update <id> --append-notes "..."`. Root-level (and stray `notes/`) summary files are agent panic-litter and will be deleted.
- **Don't close beads.** The LLM judge pipeline is the only legitimate closer. If you believe a bead is complete, leave it `in_progress` and document evidence in `--append-notes`. Closing yourself bypasses verification and tends to trigger fallback behaviors (like writing summary files) that pollute the repo. The user may explicitly authorize you to close a specific bead per-request; that authorization does not extend to others.
- **Don't run `bd edit`.** It opens `$EDITOR` and hangs the agent indefinitely. Use `bd update --description / --notes / --append-notes / --design` for content edits, and `bd update --claim / --status` for state changes.
- **Don't trust `--dry-run` with `bd create --graph`.** It is silently ignored (bd 1.0.3) and issues are created anyway. Validate plans by hand-reading the JSON and running `jq . plan.json` before applying.
- **Don't expand scope silently.** If you find an unrelated bug while working a bead, follow the Bug Discovery Protocol below — file it; only fix inline if it BLOCKS your current bead.
- **Don't reinvent prior art.** Run `bd memories` and `bd memories <keyword>` at session start. Search beads with `bd search <topic>` before filing new ones. Many "obvious" patterns and gotchas are already documented.
- **Don't display raw `dependency_type` as the relationship label.** bd reports `"blocks"` on both sides of a blocks edge; the label depends on direction AND type. See memory `dep-edge-direction` and bdboard-fjk for the full mapping.

## Bug Discovery Protocol

When you find a bug **unrelated to your current bead's stated goal**:

**One bug per bead.** Multiple unrelated issues → file each separately.

**Decide BLOCKING vs NON-BLOCKING per bug:**
- **BLOCKING** = you cannot satisfy your current bead's acceptance criteria without fixing the bug first.
- **NON-BLOCKING** = the bug exists but doesn't prevent you from completing the current bead.

**NON-BLOCKING — file and keep working:**
```bash
bd create --type=bug --priority=2 \
  --title="<short title>" \
  --description="<what you saw, repro steps, suspected cause>"
```
Then continue with your original bead.

**BLOCKING — file with triage marker, fix inline, finish work:**
```bash
# 1. File the bug with a discovered-from edge (NOT blocks).
#    Use discovered-from because after you fix inline the current bead
#    is no longer waiting on it — the bug bead exists for separate
#    verification later. A `blocks` edge here creates a self-deadlock:
#    the current bead can't close because the (intentionally) still-open
#    bug remains a blocker. discovered-from preserves traceability without
#    blocking close. (Learned the hard way via bdboard-lng ↔ bdboard-3y7.)
bd create --type=bug --priority=1 \
  --title="<short title>" \

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [weegens-aaron/bdboard](https://github.com/weegens-aaron/bdboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->

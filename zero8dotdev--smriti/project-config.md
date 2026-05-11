---
trigger: always_on
description: Shared memory layer for AI-powered engineering teams. Built on
---

# Smriti

Shared memory layer for AI-powered engineering teams. Built on
[QMD](https://github.com/tobi/qmd).

## Memory

You have Smriti. USE IT.

### Start of Every Session

Before writing a single line of code, run:

```bash
smriti recall "[what we're about to work on]"
```

Use what comes back. If there's prior context on this topic, it changes how
you approach the work. Don't ignore it.

### Recognize These Moments — Then Act Immediately

No permission needed. No asking "should I save this?" Just do it.

- Someone says "this works" or "let's go with this"
- An approach is chosen after comparing options
- A bug is fixed and the root cause is understood
- A constraint is discovered that will affect future work
- An architectural decision is made

**WRONG**: Continue the conversation without saving.
**RIGHT**: Write the decision record, ingest it, tag it — then respond.

### How to Save a Decision

**Step 1 — Write the record:**

```bash
cat > /tmp/smriti-note.md << 'EOF'
# [Title]

## Decision
[One sentence — the actual choice made.]

## Why
[The reasoning. What was considered. What was ruled out and why.]

## Context
[What problem this solves. What breaks if someone reverses this later.]
EOF
```

**Step 2 — Ingest:**

```bash
smriti ingest file /tmp/smriti-note.md \
  --title "[Title]" \
  --project smriti
```

**Step 3 — Tag:**

```bash
smriti tag [session-id] [category]
```

### Categories

| What happened | Tag |
|---------------|-----|
| Architecture choice, approach finalized | `decision/technical` |
| Library or tool selected | `decision/tooling` |
| Bug fixed, root cause identified | `bug/fix` |
| Structural design settled | `architecture/design` |
| Feature approach finalized | `feature/implementation` |
| Config or setup figured out | `project/config` |

### Concrete Example

User says: *"Okay let's use post-filtering for vector search with filters"*

**WRONG**:
> "Got it, I'll use post-filtering."

**RIGHT**:

```bash
cat > /tmp/smriti-note.md << 'EOF'
# Vector search with filters: post-filter approach

## Decision
Use post-filter strategy for applying metadata filters to vector search
results, rather than combining sqlite-vec queries with EXISTS subqueries.

## Why
sqlite-vec's two-step query pattern hangs when combined with Smriti's
EXISTS subqueries on metadata tables. Post-filtering runs vector search
unfiltered (3x overfetch), then filters results against session metadata
in a separate batch lookup — no sqlite-vec changes needed.

## Context
Affects smriti recall when --project, --category, or --agent flags are
used. Without this, filtered recall loses semantic search entirely and
falls back to BM25 only.
EOF

smriti ingest file /tmp/smriti-note.md \
  --title "Vector search with filters: post-filter approach" \
  --project smriti

smriti tag [session-id] decision/technical
```

Then respond to the user.

---

## Quick Reference

```bash
smriti ingest claude             # Ingest Claude Code sessions
smriti ingest codex              # Ingest Codex CLI sessions
smriti ingest cline              # Ingest Cline CLI sessions
smriti ingest copilot            # Ingest GitHub Copilot (VS Code) sessions
smriti ingest cursor --project-path ./myapp
smriti ingest all                # All known agents
smriti search "query"            # Hybrid search (BM25 + vector)
smriti recall "query"            # Smart recall with dedup
smriti recall "query" --synthesize  # Synthesize via Ollama
smriti list                      # Recent sessions
smriti show <session-id>         # View a session
smriti status                    # Memory statistics
smriti embed                     # Build vector embeddings
smriti categorize                # Auto-categorize sessions
smriti share --project myapp     # Export to .smriti/ for git
smriti sync                      # Import team knowledge
```

## Project Structure

```
src/
├── index.ts              # CLI entry point
├── config.ts             # Paths, env vars, defaults
├── db.ts                 # SQLite schema + Smriti metadata tables
├── qmd.ts                # Centralized re-exports from QMD package
├── format.ts             # Output formatting (JSON, CSV, CLI)
├── ingest/
│   ├── index.ts          # Orchestrator (parser -> resolver -> store)
│   ├── parsers/          # Pure agent parsers (no DB writes)
│   ├── session-resolver.ts  # Project/session resolution + incremental state
│   ├── store-gateway.ts  # Centralized ingest persistence
│   ├── claude.ts         # Discovery + compatibility wrapper
│   ├── codex.ts          # Discovery + compatibility wrapper
│   ├── cursor.ts         # Discovery + compatibility wrapper
│   ├── cline.ts          # Discovery + compatibility wrapper
│   ├── copilot.ts        # Discovery + compatibility wrapper
│   └── generic.ts        # File import compatibility wrapper
├── search/
│   ├── index.ts          # Filtered FTS search + session listing
│   └── recall.ts         # Recall with synthesis
├── categorize/
│   ├── schema.ts         # Category tree definitions
│   └── classifier.ts     # Auto-categorization (rule-based + LLM)
└── team/
    ├── share.ts          # Export knowledge to .smriti/ directory
    ├── sync.ts           # Import team knowledge from .smriti/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zero8dotdev/smriti](https://github.com/zero8dotdev/smriti) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->

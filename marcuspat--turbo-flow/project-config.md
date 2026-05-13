---
trigger: always_on
description: BRANCH=feat/search-feature
---

# CLAUDE.md — TurboFlow 4.0 / Ruflo v3.5

```
PROJECT_ID=rentamls
BRANCH=feat/search-feature
```

**Primary model: GLM-5.1** (via Coding Plan). Claude Opus on-demand for complex reasoning.
**Tech Stack:** Next.js 16.2.0, React 19, Prisma ORM, PostgreSQL (prod) / SQLite (dev), Railway

---

## BEHAVIORAL RULES

- Do what has been asked; nothing more, nothing less
- NEVER create files unless absolutely necessary — prefer editing existing files
- NEVER proactively create .md or README files unless explicitly requested
- NEVER save working files or tests to the root folder
- ALWAYS read a file before editing it
- NEVER commit secrets, credentials, or .env files
- ALWAYS use non-interactive shell flags — `cp -f`, `mv -f`, `rm -f`
- ALWAYS use `--json` flag with `bd` commands
- ALWAYS run tests before committing (if a test suite exists)
- After 3 failed approaches to the same problem — STOP and ask the human
- **NEVER merge to `main` without the Triple-Gate Merge Protocol**
- **NEVER force-push to `main` under any circumstances**
- Batch ALL related operations in a single message (todos, agent spawns, file ops, memory ops)

---

## WORK QUALITY

### Plan First
- Enter plan mode for ANY non-trivial task (3+ steps or architectural decisions)
- Write detailed specs upfront to reduce ambiguity
- If something goes sideways, STOP and re-plan — don't keep pushing a broken approach
- Use plan mode for verification steps, not just building

### Autonomous Execution
- When given a bug report: just fix it. Don't ask for hand-holding
- Point at logs, errors, failing tests — then resolve them
- Go fix failing CI tests without being told how
- Use subagents liberally to keep main context window clean — offload research, exploration, and parallel analysis

### Verification Before Done
- Never mark a task complete without proving it works
- Diff behavior between main and your changes when relevant
- Ask yourself: "Would a staff engineer approve this?"
- Run tests, check logs, demonstrate correctness

### Demand Elegance (Balanced)
- For non-trivial changes: pause and ask "is there a more elegant way?"
- If a fix feels hacky: implement the proper solution, not the quick patch
- Skip this for simple, obvious fixes — don't over-engineer
- Challenge your own work before presenting it

### Self-Improvement Loop
- After ANY correction from the human, store the lesson: `bd remember "lesson/<topic>" "what went wrong and the rule to prevent it"`
- Write rules for yourself that prevent the same mistake
- Review stored lessons at session start: `npx ruflo@latest memory search -q "lesson" --limit 10`

---

## TRIPLE-GATE MERGE PROTOCOL

Any merge into `main` (`master`/`production`/`prod`/`release`) = 3 consecutive human confirmations. No agent may merge autonomously.

```
GATE 1 — "🔒 MERGE GATE 1/3: Merging [branch] → main. [changes, risk]. Confirm?"
GATE 2 — "🔒 MERGE GATE 2/3: Tests: [pass/fail]. Conflicts: [y/n]. Confirm?"
GATE 3 — "🔒 MERGE GATE 3/3: FINAL. Type 'yes' to execute."
```

Each gate = separate turn. Non-`yes` = abort. Does NOT apply to feature-to-feature merges.

**Destructive commands** (`git reset --hard`, `rm -rf`, `prisma migrate reset`, `DROP TABLE`): one confirmation. Format: `⚠️ DESTRUCTIVE: [command]. [consequence]. Confirm?`

**Rollback:** `git revert --no-commit HEAD` → test → commit → push (skips Triple-Gate) → tell human → `bd create` the bug → `bd remember "revert/[branch]" "cause"`

**Conflicts:** Never silently auto-resolve. Simple: resolve + show. Complex: show both sides, ask human.

---

## MODEL ROUTING

**GLM-5.1 (default):** 200K context, 131K max output. No tiered routing. Generate complete files in one pass for new files >100 lines. Give full task context and let it chain steps autonomously.

**Claude Opus (on-demand):** Check `[AGENT_BOOSTER_AVAILABLE]` / `[TASK_MODEL_RECOMMENDATION]` hooks. Tier 1: Agent Booster (WASM, $0) for simple transforms. Tier 2: Haiku. Tier 3: Sonnet/Opus. Hard cap: $15/hr.

**Token optimization:** Use `--json` on all `bd` commands. Prefer `bd ready --json` over `bd list`. Use `bd prime` sparingly. When context fills: `bd compact`. Offload verbose explanations to `bd comments` and `memory store` instead of chat.

**MCP tools:** CLI tools (bd, npx ruflo, npx gitnexus) work regardless of model. If MCP server calls fail with GLM-5.1, use the CLI equivalent. Run `npx ruflo@latest doctor --fix` to check MCP registration.

---

## BEADS (bd) — Project Truth

ALL issue tracking, decisions, blockers, and discovered work goes in Beads — never in markdown TODOs.

**CRITICAL:** Never directly read/write `.beads/issues.jsonl`. Command is `bd`, NOT `beads`. Run `bd sync flush` after batch ops. Every `bd create` MUST include `--description` — self-sufficient, as if the reader has never seen your conversation.

**Core commands:**

```bash
bd ready --json                          # What's unblocked? (session start)
bd update <id> --claim --json            # Claim before starting
bd comments add <id> "progress" --json   # Record findings mid-task
bd close <id> --reason "what+why" --json # Complete (prove it works first)
bd create "Title" --description="full context" -t bug|feature|task -p 0-4 --json
bd remember "key" "value"                # Persistent knowledge

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [marcuspat/turbo-flow](https://github.com/marcuspat/turbo-flow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->

---
trigger: always_on
description: **Self-improving multi-agent orchestrator for Claude Code.** One sentence in. A team of specialists out.
---

# Nanika

**Self-improving multi-agent orchestrator for Claude Code.** One sentence in. A team of specialists out.

## Skills

Skills live under `skills/` as Go modules. Skill definitions auto-discover from `.claude/skills/`.

## Structure

```
nanika/
├── .claude/skills/          # Skill definitions (auto-discovered)
│   ├── orchestrator/        # Mission execution skill
│   └── decomposer/          # Mission decomposition skill
├── skills/                  # Skill source code (Go modules)
│   ├── orchestrator/        # Multi-agent mission orchestrator
│   └── decomposer/          # PHASE line format (knowledge-only, no binary)
├── personas/                # Persona markdown files
├── scripts/                 # generate-agents-md.sh, new-mission.sh
└── docs/                    # Skill standard, persona standard
```

## Docs

- `docs/SKILL-STANDARD.md` — Skill conventions
- `docs/PERSONA-STANDARD.md` — Persona conventions
- `.claude/skills/*/SKILL.md` — Individual skill references

## Backlog And Mission Rules

- Linear team `V` / `nanika` is the canonical backlog for status, priority, and execution queue
- runtime mission files live under `~/.alluka/missions/`
- use `~/nanika/scripts/new-mission.sh <slug>` when creating a new mission
- use the `linear` CLI for routine backlog updates

### Writing Mission Files

Always use pre-decomposed PHASE lines to get deterministic execution. Without PHASE lines, the orchestrator's LLM decomposer decides the plan — and it makes mistakes (e.g., injecting code review phases into research missions, wrong dependency ordering).

Reference the decomposer skill for the full format: `.claude/skills/decomposer/SKILL.md`

Quick reference:
```
PHASE: <name> | OBJECTIVE: <concrete deliverable> | PERSONA: <persona> | SKILLS: <skill1,skill2> | DEPENDS: <phase1,phase2>
```

OBJECTIVES must name verifiable success criteria, not directives. Vague objectives are the top driver of gate-failure retries (76% of retry waste in a 7d audit — see `~/.alluka/workspaces/20260414-9ec71068/workers/data-analyst-phase-2/retry-waste-breakdown.md`). Every retry on an Opus phase wastes $3–5.

- ❌ `OBJECTIVE: Verify the implementation and close out tracker issues`
- ✅ `OBJECTIVE: Verify by (1) running <binary> and confirming the three behaviors listed in TRK-XXX, (2) closing the issue via \`tracker update TRK-XXX --status done\`. Success = tracker list shows status=done and binary output matches the expected checklist.`

Rule of thumb: if another worker couldn't tell whether the phase succeeded without re-reading the whole codebase, the objective isn't specific enough.

### Before Running

Always dry-run first:
```bash
orchestrator run <mission-file> --dry-run --verbose
```

Verify:
- No unexpected review phases on research/non-code missions (use `--no-review` if auto-injected)
- Dependencies are correct (phases that need prior output have DEPENDS)
- Persona assignments match the work type

### Research Missions

Research missions that produce no code should use `--no-review`. The auto-injected review phase assumes code output and will review the entire repo instead of the research findings.

```bash
orchestrator run <mission-file> --no-git --no-review
```

### Code Missions

Code missions benefit from the review phase. Let it auto-inject or add explicitly:
```
PHASE: implement | OBJECTIVE: ... | PERSONA: senior-backend-engineer
PHASE: review | OBJECTIVE: Review the implementation for correctness and test coverage | PERSONA: staff-code-reviewer | DEPENDS: implement
```

## Utility Scripts

### nanika-context

`~/bin/nanika-context` (source: `plugins/orchestrator/scripts/nanika-context.sh`)

Prints a system-state snapshot to stdout for manual paste into a Claude session. Runs without spawning any missions or changing state.

```bash
nanika-context              # all sections: learnings, scheduler, tracker, nen
nanika-context learnings    # recent learnings only (cold-start quality ranking)
nanika-context scheduler    # scheduler jobs + recent failures
nanika-context tracker      # open P0/P1 tracker issues
nanika-context nen          # nen-daemon stats + shu health score
```

Sections:
- **learnings** — top 15 learnings by quality score via `orchestrator hooks inject-context`
- **scheduler** — full jobs table + any FAILED entries from the last 50 history events
- **tracker** — `tracker list --status open --priority P0/P1`
- **nen** — `nen-daemon status` + `shu query status`

### orchestrator hooks preflight

Assembles a full operational brief from all registered sections and prints it to stdout. This is the **SessionStart hook** — it runs automatically at the start of every Claude session.

```bash
# Manual invocation (same as SessionStart hook)
orchestrator hooks preflight

# Specific sections only
orchestrator hooks preflight --sections learnings,tracker

# Adjust byte budget
orchestrator hooks preflight --max-bytes 12288

# Machine-readable (no truncation)
orchestrator hooks preflight --format json

# Suppress (CI / automated sessions)
NANIKA_NO_INJECT=1 orchestrator hooks preflight
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [joeyhipolito/nanika](https://github.com/joeyhipolito/nanika) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->

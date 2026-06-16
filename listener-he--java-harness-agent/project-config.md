---
trigger: always_on
description: 1. **Anti-loop**: max 3 retries per gate/linter, max 2 for compile fixes. Exceed → STOP, ask human.
---

## Hard Rules (always apply)

1. **Anti-loop**: max 3 retries per gate/linter, max 2 for compile fixes. Exceed → STOP, ask human.

<HARD-GATE id="design-before-code">
STANDARD: do NOT Edit/Write `src/**`, `*.sql`, migrations, or `pom.xml` until `task_brief.md` §3 Allowed Scope AND §5 ACs filled AND Phase 3 Review passed.
Violation → STOP, emit `[Plan Invalidation]`, roll back to Propose.
</HARD-GATE>

<HARD-GATE id="evidence-before-archive">
Archive (any code-producing profile) requires per-AC mapping: `AC-id → test/command → output → PASS|FAIL`.
No mapping → do NOT mv to `.claude/wiki/archive/`, do NOT mark launch_spec DONE.
PATCH inline in response; STANDARD in `task_brief.md` §10 QA Evidence.
</HARD-GATE>

Full safety/commit/artifact policy: [.claude/rules/policy.md](.claude/rules/policy.md).

## Behavioral Principles

These reduce common LLM coding mistakes. They bias toward caution over speed — for trivial tasks, use judgment.

### 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

### 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

### 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

### 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

### 5. Skill Files: Reference, Not Preflight Reading

**Skills are deep-dive references, not preflight reading.** The description for each skill in your system prompt is the daily navigation — for Zone A skills (`java-architecture-standards`, `java-coding-style`, `mybatis-sql-standard`, `test-driven-development`) the description already encodes the everyday rules you need.

- Don't ritualistically read every Zone A `SKILL.md` before Implement. The description has what you need 90% of the time.
- Open `SKILL.md` only when you face a **specific technical decision** the description doesn't answer (e.g., "what's the In-Memory JOIN helper signature", "should this be a composite index leftmost-prefix or a covering index").
- During Implement, the PostToolUse hook emits a `[skill-hint]` block when a known anti-pattern slips into a file. Treat the hint as a signal to consider opening that specific `SKILL.md`, not as a blocking gate.

### 6. Past Incidents: Read When the Hook Points You at Them

Production incident facts live under `.claude/wiki/incidents/<date>_<slug>.md`. They are surfaced two ways:

- **On task start / unsure state:** run `/h-context-check` — it queries `events.jsonl` + `failure_memory.json` + `incidents/` and surfaces recent failures, in-progress task, dirty diff, related past incidents. The canonical pull-model entry; no inline auto-injection of these blocks happens otherwise.
- **Per-file lookup:** `python3 .claude/scripts/local_intel/incident_hint.py <file_path>` returns any past incident touching that file; agent invokes on demand (e.g. before deep-editing a sensitive area).

**Ingesting a new incident:** run `python3 .claude/scripts/local_intel/ingest_incident.py --help` — script saves the raw fact + prints a template; you write `.claude/wiki/incidents/<date>_<slug>.md` per the template. The `## 提醒未来 LLM` field is what every future session sees — write it well.

## Four Modes

| Mode | When | Profile (routing) | Flow |
|---|---|---|---|
| **Vibe** | TRIVIAL: read-only / explanation / cosmetic <3-line edit / `@vibe`/`@quickfix`/`@learn` shortcut / no sensitive surface in prompt | LEARN / PATCH(TRIVIAL) | Act directly. No spec, no Explorer, no WAL. |
| **Patch** | LOW: small bounded change (3–6 files), no irreversible decision, ambiguity recoverable; OR explicit `@patch` | PATCH(LOW) | **Slim Spec** (one paragraph: scope + AC) → Implement → QA → Archive. No task_brief, no WAL prompt. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [listener-He/java-harness-agent](https://github.com/listener-He/java-harness-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->

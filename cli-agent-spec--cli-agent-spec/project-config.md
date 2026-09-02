---
trigger: always_on
description: This file is for AI agents helping to **maintain and extend the specification** — adding failure modes, requirements, and schemas. It defines conventions every editor agent must follow.
---

# AGENTS.md — Spec Editor Guide for AI Agents

This file is for AI agents helping to **maintain and extend the specification** — adding failure modes, requirements, and schemas. It defines conventions every editor agent must follow.

If you are helping someone **implement** this specification in their own project, read [`IMPLEMENTING.md`](IMPLEMENTING.md) instead.

---

## Project overview

This project defines the **CLI Agent Spec** specification: a catalogue of failure modes, requirements, shared schemas, and a comparison matrix for building CLI tools that work reliably under AI agent orchestration.

Current corpus: 74 failure modes, 158 requirements.

---

## Directory structure

```
AGENTS.md                   ← spec editor guide (this file)
IMPLEMENTING.md             ← implementer guide for AI agents
comparison-matrix.md        ← solution comparison across 71 currently mapped failure modes
challenges/                 ← 74 failure modes grouped into 7 parts
  index.md                  ← master index of all failure modes
  sources.md                ← source evidence for each failure mode
  checklist.md              ← implementation checklist
  01-critical-ecosystem-runtime-agent-specific/
  02-critical-execution-and-reliability/
  03-critical-security/
  04-critical-output-and-parsing/
  05-high-environment-and-state/
  06-high-errors-and-discoverability/
  07-medium-observability/
requirements/               ← 158 requirements across 3 tiers
  index.md                  ← master index of all requirements
  f-NNN-<slug>.md           ← REQ-F: Framework-Automatic
  c-NNN-<slug>.md           ← REQ-C: Command Contract
  o-NNN-<slug>.md           ← REQ-O: Opt-In
schemas/                    ← canonical JSON Schema definitions
  index.md                  ← schema directory and codegen quick-reference
  codegen-guide.md          ← full installation + generation guide for all languages
  <name>.json               ← machine-consumable schema (source of truth)
  <name>.md                 ← human-readable companion: field table + implementation notes
```

---

## Styling rules

These rules apply to **all documents** in this project. Apply them when creating or updating any file.

### 1. No trailing periods in lists, tables, or cells

Sentences placed inside a list item, table cell, or blockquote cell MUST NOT end with a period.

**Correct**
```markdown
- Validation failed before execution began
- Safe to retry unconditionally after fixing the input

| Field  | Description              |
|--------|--------------------------|
| `code` | Stable machine-readable identifier |
```

**Incorrect**
```markdown
- Validation failed before execution began.
- Safe to retry unconditionally after fixing the input.

| Field  | Description                         |
|--------|-------------------------------------|
| `code` | Stable machine-readable identifier. |
```

Periods are only used to end sentences in **prose paragraphs** (Description, Purpose, Implementation notes body text).

### 2. Inline code for all identifiers

Flag names, field names, constants, file names, command invocations, and schema `$id` values must be wrapped in backticks.

**Correct:** `exit_codes` · `ExitCode.NOT_FOUND` · `--schema` · `error.redirect.command`
**Incorrect:** exit_codes · ExitCode.NOT_FOUND · --schema

### 3. Verb-first relationship labels in Related tables

Every row in a `## Related` table must start with a verb describing the relationship direction.

Allowed verbs: `Provides` · `Consumes` · `Enforces` · `Specializes` · `Composes` · `Aggregates` · `Wraps` · `Sources` · `Exposes` · `Extends`

### 4. Present tense in descriptions

All `description` fields (in schemas, requirement files, and table cells) use present tense.

**Correct:** `"Operation completed as intended"`
**Incorrect:** `"Operation will complete as intended"` / `"Operation has completed"`

### 5. Agent-readable `description` fields

Descriptions in `ExitCodeEntry`, `FlagEntry`, and `ErrorDetail` are read by agents at runtime. They must:
- State the condition, not the intent (`"Target cluster not found"` not `"Finds the target cluster"`)
- Be ≤ 120 characters
- Not end with a period

---

## Failure mode files

**Naming:** `{nn:02d}-{severity}-{slug}.md` inside the matching part folder

**Severity order within a part:** critical → high → medium

**Required frontmatter line** (first metadata line after the title):
```
**Severity:** Critical | **Frequency:** Common | **Detectability:** Hard | **Token Spend:** High | **Time:** High | **Context:** Medium
```

**Required sections in order:**
1. `### The Problem` — failure modes with code examples
2. `### Impact` — consequences from the agent's perspective; bulleted list, no trailing periods
3. `### Solutions` — CLI author and framework design fixes; sub-headings by audience (`**For CLI authors:**`, `**For framework design:**`); no agent-side content here
4. `### Evaluation` — scoring table (0–3) or binary pass/fail with a `**Check:**` line describing what to run or observe

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cli-agent-spec/cli-agent-spec](https://github.com/cli-agent-spec/cli-agent-spec) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-02 -->

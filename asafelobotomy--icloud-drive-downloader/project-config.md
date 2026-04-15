---
trigger: always_on
description: > **Template version**: 5.7.0 <!-- x-release-please-version --> | **Applied**: 2026-04-06
---

# Copilot Instructions — iCloud Drive Downloader

> **Template version**: 5.7.0 <!-- x-release-please-version --> | **Applied**: 2026-04-06
> Living document — self-edit rules in §8.
>
> **Models**: each `.github/agents/*.agent.md` pins its model. Codex models are headless-only (no interactive prompts). See [model comparison](https://docs.github.com/en/copilot/reference/ai-models/model-comparison).
>
> **⚡ Critical Reminders** — every session, every task:
>
> 1. **Test** — use deterministic targeted suites during intermediate phases when available; run `python3 -m pytest tests/ -v` before marking a full task done end-to-end (§3).
> 2. **PDCA** — Plan→Do→Check→Act for every non-trivial change (§5).
> 3. **Read first** — never claim or modify a file not opened this session (§4).
> 4. **Additive** — never delete existing rules without explicit user instruction (§8).

## §1 — Lean Principles

| # | Principle | This project |
|---|-----------|-------------|
| 1 | Eliminate waste (Muda) | Every line of code has a cost; every unused feature is waste |
| 2 | Map the value stream | Changes ship as updates to a portable Python CLI and supporting docs/tests that users run directly on their own machines. |
| 3 | Create flow | Most work stays inside one monolithic Python CLI plus adjacent docs/tests, so progress comes from small verified edits instead of module churn. |
| 4 | Establish pull | Build only what is needed, when it is needed |
| 5 | Seek perfection | Small, continuous improvements (Kaizen) over big rewrites |

**Waste taxonomy** (§6):

- Overproduction · Waiting · Transport · Over-processing · Inventory · Motion · Defects · Unused talent

---

## §2 — Operating Modes

Switch modes explicitly. Default is **Implement**.

### Implement Mode (default)

- Plan → implement → test → document in one uninterrupted flow.
- Full PDCA for every non-trivial change.
- Three-check ritual before marking a full task complete end-to-end.

### Review Mode

- Read-only by default. State findings before proposing fixes.
- Tag every finding with a waste category (§6).
- Use format: `[severity] | [file:line] | [waste category] | [description]`
- Severity: `critical` | `major` | `minor` | `advisory`

  <examples>
  `[critical] | [src/auth.ts:42] | [W7 Defects] | SQL query built by string concatenation — injection risk; use parameterised queries`
  `[minor] | [src/utils/format.ts:18] | [W4 Over-processing] | One-liner wrapped in a function with no added value — consider inlining`
  </examples>

#### On-Demand Review Skills

For deeper audits, activate the matching skill (§12) instead of expanding §2:

- **Extension audits** → `extension-review` skill · **Test coverage** → `test-coverage-review` skill
- Present report first; wait for approval before writing files.

### Refactor Mode

- No behaviour changes. Tests must pass before and after.
- Measure LOC delta. Flag if a refactor increases LOC without justification.

### Planning Mode

- Produce a task breakdown before writing code.
- Estimate complexity (S/M/L/XL). Flag anything XL for decomposition.

---

## §3 — Standardised Work Baselines

| Baseline | Value | Action if exceeded |
|----------|-------|--------------------|
| File LOC (warn) | 250 lines | Flag, suggest decomposition |
| File LOC (hard) | 400 lines | Refuse to extend; decompose first |
| Dependency budget | 6 runtime deps | Propose removal before adding |
| Dependency budget (warn) | 5 runtime deps | Flag for review |
| Test command | `python3 -m pytest tests/ -v` | Must pass before the full task is done |
| Type check | `python3 -m mypy icloud_downloader.py --check-untyped-defs` | Must pass before task is done |
| Three-check ritual | `python3 -m py_compile icloud_downloader.py && python3 -m mypy icloud_downloader.py --check-untyped-defs && python3 -m pytest tests/ -v` | Run before marking complete |
| Integration test gate | RUN_INTEGRATION_TESTS=1 | Set to run integration tests |
| Max subagent depth | 2 | Stop and report to user |

---

## §4 — Coding Conventions

- Language: **Python** · Runtime: **Python 3.10+** · Package manager: **pip**
- Test framework: **pytest**
- Preferred serialisation: **JSON and JSONL**

**Patterns observed in this codebase**:

- Single-file CLI architecture centered on `icloud_downloader.py`
- Security-first path validation, name sanitization, and owner-only permissions
- Thread-safe helper classes protect shared state in concurrent download flows
- Sequential and concurrent execution paths are kept semantically aligned
- Documentation, examples, and dry-run workflows are treated as first-class UX surfaces

**Universal rules**:

- No `any` / untyped unless explicitly commented with `// deliberately untyped: <reason>`.
- No silent error swallowing — log or re-throw.
- No commented-out code — git history is the undo stack.
- Imports are grouped: stdlib → third-party → internal. One blank line between groups.
- Functions do one thing. If you need "and" in the name, split it.
- Read before claiming — never describe, reference, or modify a file not opened this session.
  `semantic_search` or `grep_search` confirms existence; reading the file confirms content.

**Terminal discipline**:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/asafelobotomy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->

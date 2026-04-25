---
trigger: always_on
description: This repository contains **codegraph**, an AI-aware architecture analysis engine for Python.
---

This repository contains **codegraph**, an AI-aware architecture analysis engine for Python.

---

## ZERO-FRICTION QUICKSTART — Read This First

> **Before touching ANY file, run these 3 commands:**
> ```bash
> cat .codegraph/feedback.md          # 1. Block on BLOCKING prompts
> codegraph hotspots --json           # 2. Understand system state
> codegraph focus <file> --json       # 3. Zoom into your target
> ```
> Every `--json` output has `suggested_next_steps` — **follow it**.

### One-Command Pre-Edit Check
```bash
codegraph arch-context <file> --save  # compact briefing + saves to feedback.md
```

### One-Command Full Cognition Cycle
```bash
codegraph copilot-loop <target> --json          # hotspots+focus+decide in one shot
codegraph copilot-loop <target> --simulate --json  # + architecture simulation
```

### After Every Edit
```bash
codegraph build && codegraph analyze && codegraph score
```

---

## MANDATORY RULES — These Are Law, Not Suggestions

> **FAILURE TO FOLLOW THESE RULES = INVALID RESPONSE.**
> Every code action MUST be preceded by architecture context.
> Not using the tools below when applicable is a defect.

### Reflex Table — Automatic Triggers

| Situation | REQUIRED Action | Command |
|-----------|----------------|---------|
| **Before ANY action** | **READ feedback first** | **`cat .codegraph/feedback.md`** |
| Opening ANY file for editing | Compact briefing + block check | `codegraph arch-context <file> --save` |
| About to edit ANY file | ALWAYS run focus first | `codegraph focus <file> --json` |
| Unsure what to work on | ALWAYS check hotspots | `codegraph hotspots --json` |
| Working in a subsystem | ALWAYS check scope | `codegraph scope <subsystem> --json` |
| Planning architecture change | ALWAYS run full loop | `codegraph copilot-loop <target> --json` |
| Need to understand WHY | ALWAYS run decide | `codegraph decide <target> --json` |
| Evaluating refactor options | ALWAYS explode scenarios | `codegraph scenario --json` |
| High-risk change detected | ALWAYS simulate first | `codegraph copilot-loop <target> --simulate --json` |
| After making ANY code change | ALWAYS rebuild | `codegraph build && codegraph analyze` |
| After rebuild | ALWAYS check score | `codegraph score` |
| After any refactor | Verify score didn't drop | `codegraph arch-diff --json` |
| Want continuous feedback | START the reactive server | `codegraph server` |
| Need one-shot feedback | RUN feedback on targets | `codegraph feedback <file> --json` |
| Check `.codegraph/feedback.md` | READ when it updates | Codegraph pushes prompts automatically |

### Feedback Awareness (MANDATORY)

> **Before making changes to ANY file, READ `.codegraph/feedback.md`.**
>
> - If it contains **BLOCKING** prompts, you **MUST** resolve them first.
> - If it contains **HIGH PRIORITY** prompts, address them in your current action.
> - **DO NOT** ignore feedback prompts — they are architecture-level directives.
> - After resolving a blocking prompt, re-run `codegraph feedback <file> --json` to confirm.

### The Copilot Loop (REQUIRED execution order)

```
1. codegraph hotspots --json           # What needs attention?
2. codegraph focus <target> --json     # Zoom in on target
3. codegraph decide <target> --json    # WHY this action?
4. [make changes]                      # Implement
5. codegraph build && codegraph analyze  # Rebuild
6. codegraph focus <target> --json     # Verify improvement
7. codegraph score                     # Check score delta
```

Or use the single-command orchestrator:
```bash
codegraph copilot-loop <target> --json              # Full cognition cycle
codegraph copilot-loop <target> --simulate --json    # With simulation
codegraph copilot-loop <target> --simulate --save    # Persist decision
```

### JSON Output Guides You

Every `--json` output includes `suggested_next_steps` — an array of exact
commands to run next. **ALWAYS read and follow `suggested_next_steps`.**
The system tells you what to do next. Trust it.

### Hooks (`.claude/hooks/`)

Automated safety checks that run at lifecycle points. See `.claude/hooks/hooks.json` for configuration.

| Hook | When | Blocking |
|------|------|----------|
| `pre-edit.sh <file>` | Before any file edit | Yes — aborts if BLOCKING feedback exists |
| `post-edit.sh <file>` | After any file edit | No — rebuilds graph, checks score |
| `pre-commit.sh` | Before git commit | Yes — tests, violations, branch check |
| `pre-pipeline.sh` | Before architecture pipeline | Yes — clean tree, lock, baseline |
| `post-pipeline.sh` | After architecture pipeline | Yes — score compare, drift, lock |
| `pre-refactor.sh <target>` | Before refactoring | Yes — scope analysis, snapshot |
| `post-refactor.sh <target>` | After refactoring | Yes — build, test, score, snapshot |

Skipping a blocking hook is a rule violation.

## Self-Modification Rules (MANDATORY)

When working on **codegraph itself**, these rules prevent chaos during self-improvement:

1. **NEVER modify core subsystems without scope analysis**
   ```bash
   codegraph scope <subsystem> --json
   codegraph decide <target> --json
   ```
2. **If a BLOCKING prompt exists → STOP and resolve before continuing**
   - Read `.codegraph/feedback.md`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Donkon215/codegraph](https://github.com/Donkon215/codegraph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->

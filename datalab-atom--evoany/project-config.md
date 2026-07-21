---
trigger: always_on
description: | # | Agent | Role | Runs |
---

# U2E Evolution Protocol — Multi-Agent Architecture

## Agents

| # | Agent | Role | Runs |
|---|-------|------|------|
| 1 | **OrchestratorAgent** | Drives the main loop, dispatches workers, triggers selection | Once per run |
| 2 | **MapAgent** | Analyzes code, identifies optimization targets | Once at init |
| 3 | **WorkerAgent** | Generates a code variant (CodeGen) and evaluates it (Dev) | N per generation, in parallel |
| 4 | **PolicyAgent** | Reviews git diff, approves or rejects before benchmark | Once per worker |
| 5 | **ReflectAgent** | Writes memory, extracts lessons, runs synergy checks | Once per generation |

> **PlanAgent** is implemented server-side in `plan_generation()` — no LLM needed.

## Core Loop

The loop is driven by `evo_step`. The **OrchestratorAgent** calls it to advance
state; **WorkerAgents** call it to report code and fitness results.

```
OrchestratorAgent:
  step = evo_step("begin_generation")
  # → {action: "dispatch_workers", generation, batch_size, items: [...]}

LOOP:
  if step.action == "done":
      break

  elif step.action == "dispatch_workers":
      # Launch one WorkerAgent per item, in parallel
      for item in step.items:
          spawn WorkerAgent(item)
      wait for all workers to return

      step = evo_step("select")
      # → {action: "reflect", keep: [...], eliminate: [...], best_branch, best_obj}

      # OrchestratorAgent cleans up
      a. Delete eliminated branches
      b. Tag best: git tag best-gen-{N}

      # Hand off to ReflectAgent
      spawn ReflectAgent(step)
      step = evo_step("reflect_done")
      # → {action: "dispatch_workers", ...} or {action: "done", ...}
```

### WorkerAgent Flow (per item)

```
WorkerAgent receives: item = {branch, operation, target_id, parent_branches,
                              target_file, target_function}

1. CODEGEN — generate the variant
   a. git checkout -b item.branch from item.parent_branches[0]
   b. parent_commit = git rev-parse item.parent_branches[0]
   c. Read target function code; compute code_hash = sha256 of file content
   d. Check cache: result = evo_check_cache(code_hash=code_hash)
      If result.cached == True: skip codegen, skip benchmark, report via evo_step("fitness_ready") using cached values
   e. Read memory/ for this target (long_term + failures)
   f. Generate variant (mutate or crossover)
   g. git add + git commit

2. REQUEST POLICY CHECK
   step = evo_step("code_ready",
                    branch=item.branch,
                    parent_commit=parent_commit)
   # → {action: "check_policy", branch, diff, changed_files,
   #    target_file, protected_patterns, ...}

3. POLICY CHECK — hand to PolicyAgent
   PolicyAgent reviews step.diff:
     - Are changed_files only the declared target_file?
     - Do any changed_files match protected_patterns?
     - Was only the function body changed (not its signature)?
     - Are there hidden side effects?

   if approved:
       step = evo_step("policy_pass", branch=item.branch)
       # → {action: "run_benchmark", branch, target_id, operation, parent_branches}
   else:
       step = evo_step("policy_fail", branch=item.branch,
                        reason="<why it was rejected>")
       # → {action: "worker_done", branch, rejected=True, reason}
       return  ← worker exits early

4. BENCHMARK — evaluate the variant
   a. git worktree add <path> step.branch
   b. Run benchmark command in worktree
   c. Parse fitness from output
   d. git worktree remove <path>

   step = evo_step("fitness_ready",
                    branch=step.branch,
                    fitness_values=<list[float]>,
                    success=<bool>,
                    operation=step.operation,
                    target_id=step.target_id,
                    parent_branches=step.parent_branches)
   # → {action: "worker_done", branch, fitness_values, success, on_pareto_front, total_evals}
   return  ← worker exits
```

### ReflectAgent Flow

```
ReflectAgent receives: selection result with keep/eliminate/best_branch

1. git diff best..second_best → extract what changed
2. Write memory/targets/{id}/short_term/gen_{N}.md
3. Synthesize long_term.md from accumulated short_term
4. Record failures to memory/targets/{id}/failures.md
5. Every 3 generations: synergy check
   - Cherry-pick best of each target into one branch
   - Run WorkerAgent flow on synergy branch
   - Record results via evo_record_synergy
```

## Standalone Tools (not evo_step phases)

| Tool | Called by | Purpose |
|------|-----------|---------|
| `evo_init` | OrchestratorAgent (once) | Initialize run, set config, preload memory |
| `evo_register_targets` | MapAgent (once) | Register optimization targets |
| `evo_report_seed` | OrchestratorAgent (once) | Record seed baseline fitness |
| `evo_next_batch` | OrchestratorAgent | Request next generation batch (alternative to evo_step) |
| `evo_report_fitness` | WorkerAgent | Report individual fitness (alternative to evo_step("fitness_ready")) |
| `evo_select_survivors` | OrchestratorAgent | Run selection (alternative to evo_step("select")) |
| `evo_revalidate_targets` | OrchestratorAgent | Verify registered targets still exist in repo (file present, function defined); used after structural-op derivatives |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DataLab-atom/EvoAny](https://github.com/DataLab-atom/EvoAny) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->

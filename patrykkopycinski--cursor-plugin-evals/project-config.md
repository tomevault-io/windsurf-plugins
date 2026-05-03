---
trigger: always_on
description: >-
---


# Proactive Eval Coverage — Default Behavior

## Prime Directive

When working in ANY repository that contains a Cursor plugin (`.cursor-plugin/plugin.json`), MCP server (`mcp.json`), or skill collection (`SKILL.md` files), the assistant MUST:

1. Ensure comprehensive evaluation coverage exists
2. **Run the evals**
3. **Fix any failures**
4. **Re-run until all CI thresholds pass**

This is the DEFAULT behavior — not something the user needs to request. **The job is not done until BOTH:**

1. **`npx cursor-plugin-evals run --ci` exits 0** (eval convergence)
2. **Content audit shows no HIGH/MEDIUM issues** after up to 5 iterative passes (content convergence)

## When to Act

Act IMMEDIATELY and WITHOUT ASKING when ANY of these are true:

1. **No `plugin-eval.yaml` exists** → Generate a complete one, run it, fix failures until green
2. **Coverage score < 75/100** → Fix all gaps, run evals, iterate until thresholds pass
3. **New MCP tool added** → Write tests, run them, fix until they pass
4. **New skill added** → Write frontmatter + activation + negative tests, run, fix until pass
5. **New rule added** → Write frontmatter + content quality tests, run, fix until pass
6. **New agent added** → Write frontmatter + behavior tests, run, fix until pass
7. **New command added** → Write frontmatter + execution tests, run, fix until pass
8. **Eval results show failures** → Diagnose, fix, re-run until green
9. **CI thresholds missing** → Add them, run `--ci`, ensure exit 0
10. **Security tests missing** → Add them, run them, fix until they pass
11. **Performance tests missing** → Add benchmarks, run them, calibrate thresholds
12. **Component has no tests at all** → Generate ALL required tests for that component type
13. **Content fixes applied** → Run content audit convergence loop until no HIGH/MEDIUM remain

## The Run → Fix → Converge Loop

Every time you generate or modify eval config, you MUST run this loop:

```
REPEAT (max 5 iterations per layer):
  1. Run evals: `npx cursor-plugin-evals run [--layer X] --verbose`
  2. If all pass → DONE for this layer, move to next
  3. If failures:
     a. Read failure details
     b. Classify: config issue vs plugin bug vs infrastructure issue
     c. For config issues → fix YAML immediately (assertions, expected, thresholds)
     d. For plugin bugs → flag to user (don't modify plugin source)
     e. For infra issues → add require_env/skip with reason
  4. Go to step 1

AFTER all layers pass individually:
  Run full CI: `npx cursor-plugin-evals run --ci`
  If CI fails → identify which gate failed, fix, re-run
  Repeat until exit 0
```

### Fix Strategies by Failure Type

| Failure | Fix |
|---------|-----|
| Wrong expected tool name | Update `expected.tools` in YAML |
| Tool not registered | Add to `expected_tools` or check conditional env |
| Assertion mismatch | Update assertion to match actual response format |
| Timeout | Increase test `timeout` value |
| Missing env var | Add `require_env` to skip test when env absent |
| LLM picks wrong tool | Make prompt more specific, add hints |
| LLM wrong arguments | Update `expected.toolArgs` or relax matching |
| Security test fails | Ensure test expectation is correct (should tools NOT trigger?) |
| Flaky result | Increase `repetitions` to 3, or lower temperature |
| Score below threshold | Fix individual test failures first; relax threshold last resort |

### Convergence Safeguards

- **Max 5 iterations per layer** — if still failing, report remaining issues
- **Never remove tests** — skip with reason if infrastructure is missing
- **Steady state detection** — if same tests fail with same scores 2x in a row, change approach
- **Threshold relaxation is last resort** — fix the test first, only relax if genuinely too strict

### Post-Convergence Threshold Calibration — MANDATORY

After all CI thresholds pass, the job is NOT done. Evaluate whether the thresholds are
properly calibrated. Lenient thresholds provide a false sense of quality and allow regressions
to slip through.

**Threshold tightening rules:**

| Actual vs Threshold | Action |
|---------------------|--------|
| Actual > threshold + 20% | **MUST tighten** — bump threshold to `actual - 5%` |
| Actual > threshold + 10% | **SHOULD tighten** — bump if stable across 2+ runs |
| Actual > threshold + 5% | Well calibrated — leave as-is |
| Actual ≈ threshold | Tight — leave, monitor flakiness |
| `security.min = 1.0` | **NEVER lower** — security is absolute |

**After tightening, re-run `--ci` to confirm the new thresholds still pass.** If they fail
due to LLM variance, back off by 2% and re-run. The final committed thresholds should
pass reliably with ~5% headroom.

**Performance threshold calibration:**
- Set p50 to `actual_p50 * 1.2` (20% buffer)
- Set p95 to `actual_p95 * 1.3` (30% buffer for tail latency)
- Never use static values (200ms, 1000ms) without measuring first

### Content Audit Convergence — MANDATORY for Content Fixes

When auditing or fixing **content** (skill scripts, SKILL.md files, reference docs, shared modules),
a single scan pass is NEVER sufficient. You MUST iterate until no HIGH/MEDIUM issues remain.

#### Audit Severity Classification

| Severity | Convergence Rule | Examples |
|----------|-----------------|----------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [patrykkopycinski/cursor-plugin-evals](https://github.com/patrykkopycinski/cursor-plugin-evals) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->

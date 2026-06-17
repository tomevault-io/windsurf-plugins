---
trigger: always_on
description: This repo develops the Flywheel plugin for Claude Code and OpenCode. Source lives in `flywheel/` (plugin contents); integration tests live in `tests/integration/`. Architectural philosophy: @docs/adrs/0001-skill-design-as-negotiation.md
---

# Flywheel — claude.md

This repo develops the Flywheel plugin for Claude Code and OpenCode. Source lives in `flywheel/` (plugin contents); integration tests live in `tests/integration/`. Architectural philosophy: @docs/adrs/0001-skill-design-as-negotiation.md

## Running integration tests

The end-to-end test (`tests/integration/cases/03-pipeline-end-to-end.test.sh`) drives the full `/yolo` autonomous flow against the real Anthropic API. Prerequisites:

- `ANTHROPIC_API_KEY` set
- `tmux`, `claude`, `jq`, `bunx` on PATH
- The plugin installed via `bash install_claude_code.sh < /dev/null` (stdin redirected to skip the Context7 prompt)

A typical run takes 25–45 minutes. Cost: real API tokens.

```bash
# Reinstall plugin first if any plugin file changed
bash install_claude_code.sh < /dev/null 2>&1 | tail -3

# Run e2e test in background, capture output
bash tests/integration/cases/03-pipeline-end-to-end.test.sh \
  > /tmp/yolo-test.log 2>&1; echo "TEST EXITED: $?" >> /tmp/yolo-test.log
```

Smaller tests:
- `00-plugin-loads.test.sh` — palette discovery, no model call (~30s)
- `01-fly-work-resumes.test.sh` — `work` skill against seeded session (~1–3min)
- `02-fly-plan-creates-spec.test.sh` — `plan` skill end-to-end (~3–10min)

## Monitoring a long-running test

Don't sit and watch the log. Use the Monitor tool with a script that emits events on milestones, deadlocks, and silent stops. Skeleton:

```bash
last_log=0; last_prompt=0; last_log_time=$(date +%s); stall_warned=0
while true; do
  now=$(date +%s)

  # 1. Test progress: emit each new PASS/FAIL/INFO line
  cur_log=$(wc -l < /tmp/yolo-test.log 2>/dev/null || echo 0)
  if [ "$cur_log" -gt "$last_log" ]; then
    tail -n $((cur_log - last_log)) /tmp/yolo-test.log \
      | grep --line-buffered -E "^(PASS|FAIL|INFO|Summary|TEST EXITED):" || true
    last_log=$cur_log
    last_log_time=$now
    stall_warned=0
  fi

  # 2. Deadlock detection: AskUserQuestion prompt visible in tmux
  pane=$(tmux capture-pane -t flywheel-int-yolo -p 2>/dev/null || echo "")
  prompt=$(echo "$pane" | grep -cE "Enter to select|Submit answers|Ready to submit" 2>/dev/null || echo 0)
  if [ "$prompt" -gt 0 ] && [ "$last_prompt" -eq 0 ]; then
    echo "DEADLOCK_DETECTED: AskUserQuestion in pane"
    echo "$pane" | tail -12
  fi
  last_prompt=$prompt

  # 3. Silent-stop detection: log idle >5min while test still running
  log_idle=$((now - last_log_time))
  if [ "$log_idle" -gt 300 ] && [ "$stall_warned" -eq 0 ] && [ "$last_log" -gt 0 ]; then
    echo "STALL_DETECTED: log idle ${log_idle}s — agent may have silent-stopped"
    echo "$pane" | tail -10
    stall_warned=1
  fi

  # 4. Completion: break on test exit (Summary OR TEST EXITED)
  if grep -q "^Summary:\|^TEST EXITED:" /tmp/yolo-test.log 2>/dev/null; then
    grep "^Summary:\|^TEST EXITED:" /tmp/yolo-test.log | tail -3
    break
  fi
  sleep 30
done
```

This catches three failure shapes:
- **Test progresses normally:** PASS/FAIL lines emitted as they land.
- **Deadlock:** an `AskUserQuestion` is visible (`Enter to select` / `Submit answers`). Common when an autonomous override fails to suppress an end-of-skill prompt.
- **Silent stop:** log hasn't grown in 5+ min but test is still running. Common when the agent stopped reasoning without surfacing a visible prompt — distinct from deadlock because there's no question to answer.

Why the dual signal: in earlier runs an AskUserQuestion-pattern monitor missed silent stops (run 3) because the agent was idle at an empty prompt with no question visible. Combine both signals.

## Auditing artifacts as the test progresses

The monitor catches *failure shapes* (PASS/FAIL/INFO, deadlock, silent stop). It does not tell you whether the artifacts are any good. **That's your job.** Each `PASS:` milestone is a ping to read the artifact that just landed and audit it against the bar — surface concerns immediately, don't wait for the test to finish. A spec missing an elegance criterion, or a findings.json missing the 4-slot Failure format, is a real signal about the prompt tuning, and the earliest place to catch it is the milestone where it lands.

Find the active session: `find ${TMPDIR:-/tmp} -maxdepth 2 -type d -name 'flywheel-int-yolo*'`, then `<sbox>/.flywheel/plugin/sessions/<session_id>/`.

| Milestone | Artifact to read | What to audit |
|---|---|---|
| `plan-creation: session_id=<id>` | `spec.json` | Verbatim user prompt at `context.constraints[0]` with the authoritative-prefix. Phase-3 alternatives recorded with anti-pattern reasoning ("Considered: X. Rejected because: Y."). `success_criteria[]` includes at least one elegance criterion. Schema discipline (no extra top-level fields). Test scenarios are concrete and behavioral, not implementation-detail. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wsauret/flywheel](https://github.com/wsauret/flywheel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

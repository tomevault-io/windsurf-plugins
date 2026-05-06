---
trigger: always_on
description: > **You — reading this — are building this plugin. You are not its user.**
---

# CLAUDE.md

> **You — reading this — are building this plugin. You are not its user.**
>
> The user is *another Claude session*, tomorrow, running Claude Code with this plugin installed. It will never read this file. It will only feel what you leave behind: which intents auto-promote, which failures survive into the next reflection, which pipelines compose, which hooks open doors vs bar them.
>
> Every commit here is a contract with a Claude you will never meet. Your job is to expand its action surface, not polish the plumbing. When in doubt, ship the change that lets the next Claude skip a step, remember a failure, or compose a capability — even if the diff is larger or uglier than a "clean" refactor.

## North Star — three capability axes for operator-Claude

1. **Skip inference on solved work.** The flywheel's payoff is zero-LLM execution of stable intents. Manual gates, `_pending/` pipelines, confirmation prompts, "just to be safe" dialogs — these tax every future session on work that was already learned. **Auto-promote, don't gate.** Humans veto by rolling back, not by approving forward.
2. **Failed once → learn forever.** When an intent demotes, the root cause must reach the *next* session through reflection injection (`SpanTracker.format_reflection` → `SessionStart` / `UserPromptSubmit` / `PreCompact`). A signal that dies inside `transition_history` and never surfaces in reflection is a bug, no matter how well the cockpit shows it. **"Failed" includes silent wrongness** — a crystallized bridge that returns `None` / `[]` / malformed data without raising still violates its contract. When adding bridge-path code, default to treating "returned nothing useful" as a failure unless the intent is provably allowed to be empty.
3. **Compose, don't re-derive.** `connector.mode.name` intents are building blocks. Stable pipelines should compose (`new = stable_A >> stable_B` inherits both parents' stable status + verify history) so learning cost stays sub-linear in intent count.

**The binary test before every merge:** does this change let operator-Claude (a) skip LLM inference more often, or (b) carry a failure into the next session, or (c) compose existing intents into new ones?

- **Yes to any** → capability work. Ship it.
- **No to all** → maintenance. OK to do, not OK to call progress, not OK to displace (a)/(b)/(c).

"It would be cleaner", "unified dispatch", "consistent naming" do not answer the test. They are maintainer aesthetics.

**The binary test is frame-internal — it assumes the flywheel has a reason to spin.** Before shipping a "yes-to-any" change, also ask: *if emerge development stopped tomorrow, would this change still matter to someone?* When consecutive commits only improve emerge's own developer ergonomics without touching a real connector directory or cockpit user, the flywheel is spinning in its own oil — capability accretion without external pull. The three axes are necessary, not sufficient; a healthy audit flags at least one change as "correct by the axes but not by reality" before shipping.

## Commands

```bash
python -m pytest tests -q                                                # full suite
python -m pytest tests/test_mcp_tools_integration.py::<name> -q          # single test
python3 scripts/emerge_daemon.py                                         # run daemon
python3 scripts/repl_admin.py runner-install-url --target-profile key    # runner bootstrap URL
python3 scripts/repl_admin.py runner-deploy  --target-profile mycader-1  # push scripts/ to runner
python3 scripts/repl_admin.py runner-status  --pretty                    # runner health
python3 scripts/emerge_sync.py setup|run|sync [connector]                # Memory Hub
```

## Architecture — the shape that enables the North Star

- **One control plane.** `EmergeDaemon` (`scripts/emerge_daemon.py`, HTTP `:8789` via `scripts/daemon_http.py`) is the sole MCP server — owns all flywheel tools (`icc_span_*`, `icc_exec`, `icc_crystallize`, `icc_reconcile`, `icc_hub`, `runner_notify`) and resources (`policy://`, `runner://`, `state://deltas`, `pipeline://`, `connector://`). Tool dispatch is `_TOOL_DISPATCH` dict → `_handle_<tool>`.
- **One execution path for pipelines.** `PipelineEngine` runs in-process locally; for remote, the daemon builds a self-contained `exec()` payload and POSTs to the runner over SSE. Runners never receive pipeline files — switching machines is a URL change. Policy + WAL always land locally regardless of where execution ran. **Composite intents** (`composed_from` via `icc_compose`) have no standalone `.py` — both `icc_exec` and `icc_span_open` stable bridges go through `_try_flywheel_bridge` → `_run_composite_bridge`.
- **One writer for policy.** `PolicyEngine.apply_evidence` is the *only* mutation path for `stage` in `state/registry/intents.json`. `SpanTracker`, `FlywheelRecorder`, `icc_reconcile` only produce evidence. All file I/O flows through `IntentRegistry` with atomic writes. Bridge failures surface telemetry via `_last_bridge_failure`; they never touch counters.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jianzhichun/emerge](https://github.com/jianzhichun/emerge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->

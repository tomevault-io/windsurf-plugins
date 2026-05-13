---
trigger: always_on
description: TypeScript CLI tool that enforces browser-based verification for AI coding agents.
---

# IronBee CLI — Project Guide

## What This Is
TypeScript CLI tool that enforces browser-based verification for AI coding agents.
When an agent edits code, it cannot complete its task until it verifies the changes
work in the browser via the IronBee browser-devtools MCP server.

Core principle: **Code without verification is incomplete work.**

## Commands
```
ironbee install [project-dir] [--client <name>]   Install hooks and guidance files
ironbee uninstall [project-dir]                   Remove hooks, guidance files, and .ironbee/
ironbee update                                    Update IronBee CLI to the latest version
ironbee status  [project-dir]                     Show verdict status for all active sessions
ironbee verify  [session-id]                      Dry-run verdict validation (Stop hook checks)
ironbee hook session-start                        SessionStart hook — records session start
ironbee hook session-end                          SessionEnd hook — records session end with reason
ironbee hook activity-start                       UserPromptSubmit/beforeSubmitPrompt hook — starts activity tracking
ironbee hook require-verification                 PreToolUse hook — blocks browser tools until verification-start
ironbee hook require-verdict                      PreToolUse hook — blocks edits until verdict submitted
ironbee hook clear-verdict                        PostToolUse hook — called by client automatically
ironbee hook track-action                         PostToolUse + PostToolUseFailure — queues every tool_call as a send_event
ironbee hook verify-gate                          Stop hook — called by client automatically
ironbee hook verification-start                   Start verification cycle (called by agent via Bash)
ironbee hook verification-end                     End verification cycle (called by agent via Bash)
ironbee hook submit-verdict                       Submit verdict (called by agent via Bash, session_id in JSON)
ironbee analyze [session-id]                      Analyze session metrics (single or all sessions)
ironbee enable-backend <runtime>  [--client <n>]  Enable backend verification for a runtime (writes opinionated default verifyPatterns); --client narrows md updates to one client
ironbee disable-backend <runtime> [--client <n>]  Disable backend verification (resets verifyPatterns to []; preserves customizations); --client narrows md updates to one client
ironbee enable-verification       [--client <n>]  Turn enforcement on (default state); --client narrows artifact rerender to one client
ironbee disable-verification      [--client <n>]  Monitoring-only mode (no enforcement; sessions/tools still ship to collector); --client narrows artifact rerender to one client
ironbee queue status  [--session <id>]            Queue status per session (counts, recent dead-letter errors)
ironbee queue drain   [--session <id>]            Synchronously drain pending snapshots
ironbee queue dead-letter list  [--limit N] [--session <id>]
ironbee queue dead-letter stats [--session <id>]  Histogram by category
ironbee queue dead-letter retry <job_id> [--session <id>]
ironbee queue dead-letter clear [--session <id>]
ironbee queue purge --snapshots                   (Dangerous) delete all snapshots without processing
ironbee queue purge --sessions older-than=<dur>   Remove old empty queue/ subdirs (e.g. 14d, 2h)
ironbee process-job-file <path>                   Worker entry — process one snapshot (spawned detached)
```

## Project Structure
```
src/
├── commands/
│   ├── install.ts        # client dispatcher — auto-detects or uses --client flag
│   ├── uninstall.ts      # ironbee uninstall — removes hooks, guidance, .ironbee/
│   ├── update.ts         # ironbee update — self-update via npm
│   ├── hook.ts           # ironbee hook [session-start|session-end|activity-start|require-verification|require-verdict|clear-verdict|track-action|verify-gate|verification-start|verification-end|submit-verdict]
│   ├── status.ts         # ironbee status — lists .ironbee/ session verdicts
│   ├── verify.ts         # ironbee verify — dry-run verdict validation
│   ├── analyze.ts        # ironbee analyze — session metrics (time + verification quality)
│   ├── queue.ts          # ironbee queue [status|drain|dead-letter {list,stats,retry,clear}|purge]
│   ├── backend-toggle.ts # shared helper: applyEnableBackend / applyDisableBackend / knownRuntimes (consumed by enable/disable-backend.ts)
│   ├── enable-backend.ts # ironbee enable-backend <runtime> — opt-in toggle for backend runtime cycles (writes default verifyPatterns + flips runtime markers in installed md files)
│   ├── disable-backend.ts # ironbee disable-backend <runtime> — resets verifyPatterns to []; preserves customizations
│   ├── verification-toggle.ts # shared helper: applyVerificationToggle (artifacts-before-config ordering); consumed by enable/disable-verification.ts
│   ├── enable-verification.ts # ironbee enable-verification — flips verification.enable: true + re-renders client artifacts
│   ├── disable-verification.ts # ironbee disable-verification — flips verification.enable: false (monitoring-only mode)
│   └── process-job-file.ts # ironbee process-job-file <path> — detached worker entry
├── analysis/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ironbee-ai/ironbee-cli](https://github.com/ironbee-ai/ironbee-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->

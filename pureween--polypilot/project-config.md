---
trigger: always_on
description: Before implementing new session lifecycle, orchestration, watchdog, or event handling code, check the Copilot SDK API surface. The `copilot-sdk-reference` skill has the complete v0.2.2 API reference (453 types, 76 events, 6 hooks, 16 RPC APIs). The `processing-state-safety` and `multi-agent-orchestration` skills each contain SDK migration matrices for their domains. Prefer SDK APIs over custom implementations. When custom code is necessary, add a `// SDK-gap: <reason>` comment explaining why.
---

# PolyPilot — Copilot Instructions

## SDK-First Development

Before implementing new session lifecycle, orchestration, watchdog, or event handling code, check the Copilot SDK API surface. The `copilot-sdk-reference` skill has the complete v0.2.2 API reference (453 types, 76 events, 6 hooks, 16 RPC APIs). The `processing-state-safety` and `multi-agent-orchestration` skills each contain SDK migration matrices for their domains. Prefer SDK APIs over custom implementations. When custom code is necessary, add a `// SDK-gap: <reason>` comment explaining why.

### SDK Update Audit (run when bumping `GitHub.Copilot.SDK` version)

When the user says **"Run the SDK update audit"** or updates the SDK NuGet package version:

1. **Diff the XML docs** to find new/removed/changed types:
   ```bash
   diff <(grep 'member name="T:' ~/.nuget/packages/github.copilot.sdk/OLD_VERSION/lib/net8.0/GitHub.Copilot.SDK.xml | sort) \
        <(grep 'member name="T:' ~/.nuget/packages/github.copilot.sdk/NEW_VERSION/lib/net8.0/GitHub.Copilot.SDK.xml | sort)
   ```
2. **Update `.claude/skills/copilot-sdk-reference/SKILL.md`** — add new types, events, RPC APIs, hooks, SessionConfig properties. Update version number and version history table.
3. **Re-check migration matrices** in `processing-state-safety` and `multi-agent-orchestration` skills — any "Not adopted" items now have better SDK support? Any new APIs that replace custom code?
4. **Update `.github/copilot-instructions.md`** — SDK Event Flow, SDK Data Types, and any behavioral claims that changed.
5. **Check for JS-only features that moved to .NET** — especially `AgentStop`/`SubagentStop` hooks.
6. **Create a PR** with all changes and run the multi-model review process.

## Build & Deploy Commands

### Mac Catalyst (primary dev target)
```bash
PolyPilot/relaunch.sh      # Build + async hot-relaunch (ALWAYS use this after code changes)
PolyPilot/relaunch.sh --sync  # Build + blocking relaunch (for interactive terminal use only)
dotnet build -f net10.0-maccatalyst   # Build only
```

> **Run relaunch.sh from YOUR worktree**, not from `~/Projects/AutoPilot/PolyPilot/`.
> The script is tracked in git at `PolyPilot/relaunch.sh` and uses `dirname "$0"` to
> resolve its build directory, so each worktree's copy builds its own code.

#### ⚠️ Relaunch from a Copilot agent session

`relaunch.sh` is **async by default** — it returns immediately after a successful build, then kills the old UI and launches the new one in a detached background process after a 10-second delay. This is critical because PolyPilot hosts the Copilot sessions via TCP to the persistent CLI server. If the script blocked and killed the UI synchronously, the TCP connection would drop mid-tool-call and the agent's turn would be interrupted.

> **🚨 RULES FOR CALLING RELAUNCH.SH 🚨**
> 1. Do NOT chain ANYTHING after `./relaunch.sh` in the same bash call — no `&&`, `;`, `|`, `sleep`.
> 2. You have ~10 seconds after relaunch.sh returns to make additional quick tool calls
>    (e.g., `maui devflow wait`, `tail`, short verification commands). Use this window
>    to keep working — verify the relaunch, reconnect to MauiDevFlow, test your changes.
> 3. If one tool call gets interrupted by the kill, that's OK — the CLI keeps your session
>    alive. Just continue on the next turn.
> 4. Do NOT make long-running tool calls (>8s) after relaunch — they will be interrupted.

**Correct pattern — keep working after relaunch:**
```bash
# Tool call 1: relaunch (returns immediately after build)
PolyPilot/relaunch.sh
```
After relaunch.sh returns, the old UI will be killed in ~10s and a new one launched.
Your turn may get interrupted if a tool call is in-flight when the kill happens — that's OK,
the CLI keeps your session alive. On your next turn, verify and continue:
```bash
# Next turn: verify relaunch + reconnect to MauiDevFlow
tail -3 ~/.polypilot/relaunch.log && maui devflow wait --timeout 30
```
```bash
# Then test your changes via CDP
maui devflow cdp Runtime evaluate '...'
```

**NEVER do this:**
```bash
# ❌ WRONG — chaining in the same bash call blocks the tool return
PolyPilot/relaunch.sh && sleep 15 && cat ~/.polypilot/relaunch.log

# ❌ WRONG — sleep/long commands chained after relaunch
PolyPilot/relaunch.sh; sleep 10; tail ~/.polypilot/relaunch.log
```

The `--sync` flag restores the old blocking behavior (for human terminal use only — NEVER use from an agent).

### Tests
```bash
cd ../PolyPilot.Tests && dotnet test              # Run all tests
cd ../PolyPilot.Tests && dotnet test --filter "FullyQualifiedName~ChatMessageTests"  # Run one test class
cd ../PolyPilot.Tests && dotnet test --filter "FullyQualifiedName~ChatMessageTests.UserMessage_SetsRoleAndType"  # Single test
```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PureWeen/PolyPilot](https://github.com/PureWeen/PolyPilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->

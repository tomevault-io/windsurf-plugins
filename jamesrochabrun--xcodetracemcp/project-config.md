---
trigger: always_on
description: This repository contains a local Model Context Protocol (MCP) server for Xcode Instruments profiling. Treat it as an honest `xcrun xctrace` companion, not a replacement for Instruments.app. The server records, opens saved traces in Instruments.app by default, symbolicates, exports, parses, and analyzes what Apple exposes through `xctrace`, then reports unsupported or non-exportable data explicitly.
---

# Agent Guide

This repository contains a local Model Context Protocol (MCP) server for Xcode Instruments profiling. Treat it as an honest `xcrun xctrace` companion, not a replacement for Instruments.app. The server records, opens saved traces in Instruments.app by default, symbolicates, exports, parses, and analyzes what Apple exposes through `xctrace`, then reports unsupported or non-exportable data explicitly.

## Project Architecture

The repo is a pnpm workspace with two packages:

- `packages/core`: reusable TypeScript library for `xcrun xctrace` capability checks, recording/exporting, symbolication, trace parsing, analysis, recommendations, and Time Profiler comparisons.
- `packages/mcp-server`: MCP stdio server that exposes the core library as assistant-callable tools.

High-level flow:

1. An MCP client calls a tool such as `profile_running_app`.
2. The MCP server validates arguments and delegates to `@xctrace-analyzer/core`.
3. Core runs `xcrun xctrace` using `execFile` argument arrays.
4. Core parses `xctrace export --toc`, TOC-driven XPath table XML, and HAR output when available.
5. Core returns typed analysis objects with support status and export attempts.
6. The MCP server formats those objects into Markdown, JSON, or both.

## User-Facing Skill

This repo includes a bundled skill at `skills/xctrace-profiler`. Use it as the primary human-facing entry point for profiling, trace analysis, hangs, CPU bottlenecks, leaks, allocations, network, energy, startup slowness, and trace comparisons.

Users should be able to say simple prompts like:

```text
Profile this app.
Find why this app is slow.
Profile this app for hangs.
Profile this app for hangs and tell me which of my code is responsible.
Check this build for leaks and allocation churn.
Analyze network activity.
Launch this app and profile startup hangs.
Analyze this trace.
Compare these two traces.
```

The skill should hide MCP JSON and tool names. It is the planning layer. It chooses between `profile_running_app`, `track_running_app`, `analyze_trace`, `compare_traces`, `check_xctrace`, `list_templates`, and `list_devices`, records or analyzes with `outputFormat: "both"` when diagnostics matter, reads Support Matrix / Export Diagnostics, and can rerun `analyze_trace` with `timeRangeMs` around the longest hang so `## Top User-Code Frames` answers the app-owned code question.

## MCP Tools

When driving the MCP from another app repository, a user should be able to start with a simple prompt:

```text
Profile this app.
```

Recommended workflow:

1. For normal user prompts, use the bundled `xctrace-profiler` skill as the planning layer.
2. If the app is already running, prefer `profile_running_app` with a PID in `processName`, especially when several processes share the same name.
3. Use `outputFormat: "both"` while validating a workflow so the response includes Markdown plus structured `supportStatus` and `exportAttempts`.
4. Use launch mode only when startup behavior is the target. If the trace is saved but `xctrace export --toc` fails with `Document Missing Template Error`, treat the run as not exportable and retry with attach-by-PID.
5. For hangs, record for long enough to reproduce the issue, then inspect `## Hangs`, Support Matrix, and Export Diagnostics before drawing conclusions from "no issues" summaries.
6. If `## Hangs` reports no exported events, interpret that as trace-window scoped. It does not rule out startup or interaction hangs outside the captured window.
7. Interpret `partial` as "some usable rows parsed" and `not_exportable` as "Xcode exposed schemas but exported no usable rows." Do not treat `not_exportable` as evidence of no issues.
   `not_exportable` can also mean the Instruments.app GUI track exists but `xcrun export --toc` exposes no table schema for it.
8. If Time Profiler reports "failed to parse," treat CPU samples as unavailable and inspect Export Diagnostics; the trace may still contain usable Hangs, Memory, Network, Allocations, or Leaks data.
9. After identifying a hang start and duration, rerun `analyze_trace` with `timeRangeMs: { startMs, endMs }` around that interval to answer "what ran during the hang?"
10. Use `## Top User-Code Frames` to answer "which of my code was slow?" Pass `userBinaryHints` if module names do not match the process names discovered from the TOC.
11. Recording tools open saved traces in Instruments.app by default. Use `openInInstruments: false` only for CI or headless automation.
12. Do not delete a trace immediately after reporting while the user may still need Instruments.app verification. Once the user says the trace is no longer needed, call `cleanup_traces` with exact `tracePaths` and `dryRun: false`. For broad directory cleanup, preview first or require `olderThanMinutes`.

## Security Considerations


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jamesrochabrun/XcodeTraceMCP](https://github.com/jamesrochabrun/XcodeTraceMCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->

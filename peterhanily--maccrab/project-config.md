---
trigger: always_on
description: swift build                    # Debug build
---

# CLAUDE.md -- MacCrab Development Guide

## Build Commands

```bash
swift build                    # Debug build
swift build -c release         # Release build
make dev                       # Build + codesign + compile rules + restart daemon
make dev-no-es                 # Dev without sudo (no Endpoint Security)
make build                     # Build only (no start)
make compile-rules             # Compile YAML rules to JSON
```

## Test Commands

```bash
swift test                     # Unit tests (2165 tests in 403 suites)
make test                      # Unit tests (summary only)
make test-full                 # Full test suite
make test-integration          # Integration test (starts daemon, triggers actions)
make test-detection            # Detection coverage test (15 categories)
make test-campaign             # Multi-tactic kill chain simulation (5 waves)
make test-fp                   # False positive test (~50 system processes)
make test-stress               # Sustained operation monitor (60s default)
make lint-rules                # Rule linting
```

## Architecture

MacCrab is a local-first macOS threat detection engine. Since v1.3 (April 2026), the detection engine ships as a native Endpoint Security **System Extension** activated from inside `MacCrab.app`. Eight SPM targets:

- **MacCrabCore** (`Sources/MacCrabCore/`) -- Shared library: detection engines, collectors, enrichment, storage, prevention
- **MacCrabAgentKit** (`Sources/MacCrabAgentKit/`) -- Shared daemon bootstrap wrapping the event loop, monitors, timers, and signal handlers. Linked by both the sysext and the legacy standalone daemon
- **MacCrabAgent** (`Sources/MacCrabAgent/`) -- System Extension executable. Wrapped into `com.maccrab.agent.systemextension` bundle by `scripts/build-release.sh` and activated via `OSSystemExtensionRequest`. Ships in release DMGs
- **maccrabd** (`Sources/maccrabd/`) -- Legacy standalone daemon. Kept for `swift run maccrabd` development when no ES entitlement is available — falls back through `eslogger` → `kdebug` → FSEvents
- **MacCrabForensics** (`Sources/MacCrabForensics/`) -- Mac Context Plugin Platform: forensic case/collector/plugin library. Linked by `maccrabctl`, `MacCrabApp`, and `maccrab-mcp`; intentionally not linked by the sysext or `maccrabd`
- **maccrabctl** (`Sources/maccrabctl/`) -- CLI tool for status, events, alerts, threat hunting, reports
- **maccrab-mcp** (`Sources/maccrab-mcp/`) -- MCP server exposing ~78 tools for AI agent integration (v1.10 trace tools, v1.12.0 supply-chain / intent tools, and `forensics.*` plugin tools)
- **MacCrabApp** (`Sources/MacCrabApp/`) -- SwiftUI menubar app + dashboard + SystemExtension activator. Reads from the engine's SQLite DB

### Key Directories

```
Sources/MacCrabCore/
  Events/         Unified event model: Event, EventEnums, ProcessInfo, FileInfo, NetworkInfo, TCCInfo
  Models/         Alert model and other top-level data types
  Collectors/     Event sources (ES, Unified Log, network, DNS, TCC, EDR monitor, etc.)
  Detection/      Rule engine, sequence engine, baseline, campaign detector, behavior scoring, response actions
  Enrichment/     Process lineage, code signing, threat intel, CDHash, cert transparency, file hasher
  Prevention/     DNS sinkhole, network blocker, persistence guard, response-action safety validators
  Fleet/          Fleet telemetry client and data models
  AIGuard/        AI coding tool monitoring (AIToolRegistry, MCPAttributor, AgentLineageService)
  LLM/            LLM backends (Ollama, Claude, OpenAI, Mistral, Gemini), prompts, cache, sanitizer
  Storage/        SQLite event/alert/campaign stores, schema migrator, suppressions, encryption
  Output/         Notifications, webhooks, syslog, reports, OCSF mapper, S3, SFTP, stream sinks
  Network/        SecureURLSession (TLS 1.2 floor, optional SPKI pinning) used by all outbound HTTP
  Deception/      Honeyfile manager (canary credential paths)
  Utilities/      LockedCounter, PowerGate (battery/thermal gating), shared primitives
  Integrations/   SecurityToolIntegrations (CrowdStrike, SentinelOne log ingestion)

Rules/            436 single-event Sigma-compatible YAML rules (19 tactic directories)
  sequences/      41 multi-step sequence rules
  graph/          6 multi-entity TraceGraph rules (v1.12.0)
Compiler/         Python rule compiler (YAML -> JSON) with duplicate key and field validation
fleet/            Python fleet collector server
scripts/          Build, test, install, red team simulation, and CI scripts
Tests/            Swift Testing unit tests (2165 tests in 403 suites)
```

## Detection Stack (5 tiers)

1. **Rules** -- 436 single-event Sigma-compatible YAML rules compiled to JSON predicates, plus 41 sequence rules across 19 tactic dirs, plus 6 graph rules (Rules/graph/*.json) evaluated against materialized TraceGraph traces. Category-indexed for O(1) dispatch. Rules >50ms logged for profiling.
2. **Anomaly** -- Welford z-score statistical anomaly; 2nd-order Markov chain process trees; behavioral scoring (70+ weighted indicators with feedback-adjusted weights).
3. **Sequences** -- 41 temporal multi-step rules with process lineage correlation, 10K partial match cap.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [peterhanily/maccrab](https://github.com/peterhanily/maccrab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

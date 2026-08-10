---
trigger: always_on
description: REA exposes reverse-engineering tools through a CLI and MCP server. Hopper and the bring-your-own Ghidra adapter are operation-capable deep binary-analysis providers. Ghidra is supported on Linux x64 and has an experimental Windows x64 P0 boundary for approved native x86-64 PE applications; it supplies admitted read-only inventory and function-analysis operations but no GUI or mutation authority. Keep provider-specific code out of the domain and application layers.
---

# Repository Guidelines

## Product Direction

REA exposes reverse-engineering tools through a CLI and MCP server. Hopper and the bring-your-own Ghidra adapter are operation-capable deep binary-analysis providers. Ghidra is supported on Linux x64 and has an experimental Windows x64 P0 boundary for approved native x86-64 PE applications; it supplies admitted read-only inventory and function-analysis operations but no GUI or mutation authority. Keep provider-specific code out of the domain and application layers.

Prioritize:

- tool results that distinguish observations, inferences, and unknowns;
- equivalent behavior through the CLI and MCP;
- additive, idempotent configuration with backups;
- end-to-end tests for packaged artifacts and real Hopper/Ghidra claims.

Installers must not install or upgrade Homebrew, Node.js, npm, Java, Ghidra, or other unrelated software. Ghidra is bring-your-own. `rea setup` must print its planned changes and require approval before writing files or installing Hopper.

REA is a local-only tool; do not sanitize actionable local diagnostics such as artifact paths, digests, mismatch locations, or analysis metadata, while continuing to redact genuine secrets such as credentials and authorization headers.

## Project Structure & Module Organization

REA is a layered ESM TypeScript application. Dependencies flow inward: `domain` underlies `contracts` and the shared `process` primitives; providers depend on those layers, followed by `application`, `server`, and the entry adapters.

See [docs/architecture.mermaid](docs/architecture.mermaid) for a visual architecture diagram.

- `scripts/rea.mjs`: executable dispatcher. Routes only bare `mcp` and `--mcp` to the production stdio server; Incur handles registration utilities and one-shot commands.
- `src/main.ts`: MCP adapter. Parses config, wires the shared session runtime, starts stdio transport, and owns process-lifetime shutdown.
- `src/cli.ts`: one-shot CLI adapter for setup, diagnostics, analysis, and decompilation.
- `src/config.ts`: Zod-validated parsing of environment configuration into `AppConfig`.
- `src/domain/`: pure, side-effect-free modules. `errors.ts` owns the tagged error algebra; `result.ts` owns `Result`/`ok`/`err`; `hopperValues.ts` owns shared function-dossier values plus Hopper boundary parsers; `symbolAnalysis.ts` parses Swift/ObjC names; `javascriptApplicationGraph.ts` validates and canonically commits the provider-neutral JavaScript Application Graph; `javascriptStaticAnalysis.ts` performs bounded AST-only JavaScript structure recovery.
- `src/contracts/`: caller-visible tool schemas and catalog metadata; `toolContracts.ts` owns the canonical inventory and `enhancedInputs.ts` owns enhanced input parsing.
- `src/process/`: provider-neutral process ownership and lifecycle primitives. It owns private runtime roots, session-assigned run identity and token-verified lineage, absolute startup deadlines, correlated request waits, bounded output capture, and TERM-to-KILL cleanup without defining any provider wire protocol.
- `src/replay/`: Linux x64 controlled-JavaScript-replay adapter. It owns exact runtime closure inspection, Bubblewrap/seccomp/cgroup admission, the disposable worker, strict parent/worker protocol validation, and complete cleanup observation.
- `src/browser/`: loopback CDP/Inspector discovery, bounded WebSocket transport, exact-origin or canonical-root target authorization, passive browser/Electron and attach-only Node/Electron V8 observation, plus controlled Playwright scenarios with explicit launch/attach ownership and cleanup.
- `src/hopper/`: Hopper launch and Unix-socket protocol mechanics. `BridgeLauncher.ts` spawns the Hopper app with the in-process bridge, `HopperClient.ts` correlates request/response over the socket with timeouts and cancellation, `protocol.ts` frames bridge messages.
- `bridge/hopper_bridge.py`: runs inside Hopper and adapts declared operations to Hopper's public Python API. Hopper's bundled MCP server is not used.
- `src/ghidra/`: exact Ghidra 12.1.2/JDK 21 inspection, analysis-profile commitment, digest-bound target snapshots, isolated `analyzeHeadless` launch, authenticated Unix-socket or Windows loopback transport, bounded serial request queue, and strict inventory/function boundaries.
- `src/dotnet/`: execution-free managed PE/CLI inspection. It owns bounded PE, CLI metadata, heap, table, and resource parsing for `rea-dotnet-static`; it must never load, reflect, execute, decompile, or resolve target assemblies.
- `bridge/ghidra/ReaGhidraBridge.java`: packaged read-only `HeadlessScript` loaded through Ghidra's `scriptPath`; it owns the persistent decompiler and adapts admitted inventory, function, reference, and CFG operations to public Ghidra APIs.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [morluto/rea](https://github.com/morluto/rea) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->

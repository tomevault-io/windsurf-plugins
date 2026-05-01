---
trigger: always_on
description: generates the JSON schema, parameter parsing, and output encoding.
---

# CLAUDE.md

Guidance for AI coding assistants (Claude Code, Cursor, etc.) working in this
repository. This file is the canonical, in-repo briefing — read it before making
changes.

## What is Swarm?

Swarm is a Swift 6.2 framework for building **agents and multi-agent
workflows** on Apple platforms (iOS 26+, macOS 26+, tvOS 26+) and Linux. It is
built around:

- **Agents** — `Agent` struct with `@ToolBuilder` trailing closures, an
  `AgentRuntime` protocol, and pluggable inference providers.
- **Workflows** — fluent composition (`.step`, `.parallel`, `.route`,
  `.repeatUntil`) compiled to a DAG with checkpoint/resume.
- **Tools** — the `@Tool` macro generates JSON schemas from Swift structs at
  compile time; `FunctionTool` covers ad-hoc closures.
- **Memory** — conversation, sliding-window, summary, vector, and
  persistent backends.
- **Guardrails / Resilience / Observability** — first-class concerns, not
  bolt-ons.
- **Providers** — Foundation Models, Anthropic, OpenAI, Ollama, Gemini,
  OpenRouter, MLX, all routed through [Conduit](https://github.com/christopherkarani/Conduit).
- **MCP** — Model Context Protocol client and server support.

The package uses Swift 6.2 with `StrictConcurrency` enabled across all targets.
**All public types must be `Sendable`** — the compiler enforces it.

## Repository Layout

```
Swarm/
├── Package.swift                  # SPM manifest (Swift 6.2, traits, products)
├── README.md                      # User-facing overview
├── Sources/
│   ├── Swarm/                     # Main library (156 .swift files)
│   │   ├── Agents/                # Agent struct, workspace integration
│   │   ├── Core/                  # AgentRuntime, Conversation, Environment,
│   │   │                          #   PromptEnvelope, RuntimeMetadata, …
│   │   ├── Workflow/              # Workflow + durable engine + checkpointing
│   │   ├── Tools/                 # Tool protocol, ToolCollection, ParallelExecutor,
│   │   │                          #   built-ins, web tools, schema bridging
│   │   ├── Memory/                # Conversation, sliding window, summary, vector,
│   │   │                          #   SwiftData, ContextCore, hybrid backends
│   │   ├── Providers/             # Conduit adapters, multi-provider, sessions
│   │   ├── Guardrails/            # Input/Output/Tool guardrail specs + runner
│   │   ├── Resilience/            # Retry, circuit breaker, fallback, rate limit
│   │   ├── Observability/         # AgentTracer, SwiftLog/OSLog tracers, metrics
│   │   ├── MCP/                   # MCPClient, MCPServer, ToolBridge, capabilities
│   │   ├── Workspace/             # AgentWorkspace (AGENTS.md, .swarm/ skills)
│   │   ├── Macros/                # Public macro declarations
│   │   ├── Integration/           # Membrane and Wax integrations
│   │   └── Internal/GraphRuntime/ # Compiled DAG runtime (internal)
│   ├── SwarmMacros/               # Compiler plugin (@Tool, @Parameter,
│   │                              #   @Traceable, #Prompt, builders)
│   ├── SwarmMembrane/             # Membrane workflow integration product
│   ├── SwarmMCP/                  # MCP server adapter product
│   ├── SwarmCapabilityShowcase/        # Executable: deterministic showcase CLI
│   ├── SwarmCapabilityShowcaseSupport/ # Library backing the showcase
│   ├── SwarmDemo/                 # (opt-in) demo executable
│   └── SwarmMCPServerDemo/        # (opt-in) MCP server demo
├── Tests/
│   ├── SwarmTests/                # Main test target (mirrors Sources/Swarm)
│   │   └── Mocks/                 # MockAgentRuntime, MockInferenceProvider, …
│   ├── HiveSwarmTests/            # Hive integration tests (gated)
│   ├── SwarmMacrosTests/          # Macro expansion tests
│   └── SwarmCapabilityShowcaseTests/
├── Examples/CodeReviewer/         # Standalone example SPM project
├── docs/
│   ├── guide/                     # Getting started, agent workspace, showcase
│   ├── reference/                 # API catalog, front-facing-api, audits
│   └── release/                   # release-checklist.md
└── .github/workflows/             # swift.yml, claude.yml, claude-code-review.yml,
                                   #   docs.yml
```

## Build, Test, Lint

This is a Swift Package — there is no Xcode project committed. All commands run
from the repo root.

```bash
swift package resolve         # Resolve dependencies
swift build                   # Build the library targets
swift test                    # Run all tests
swift test --no-parallel      # Match CI ordering (recommended)
swift test --filter SwarmTests.WorkflowTests   # Run a single suite
```

CI (`.github/workflows/swift.yml`) runs on macOS 15 and Ubuntu with Swift 6.2
and uses these env vars:

```bash
SWARM_HIVE_RUNTIME=1 SWARM_INCLUDE_HIVE=1 swift test --no-parallel
```

The Hive integration tests live in the `HiveSwarmTests` target. The
`orchestrationRequireHiveFailClosed` test is asserted explicitly in CI as a
fail-closed contract.

### Demo / benchmark executables

The `SwarmDemo`, `ContextBenchmark`, and `SwarmMCPServerDemo` executables are
**opt-in** — they only build when `SWARM_INCLUDE_DEMO=1` is set:

```bash
SWARM_INCLUDE_DEMO=1 swift build
SWARM_INCLUDE_DEMO=1 swift run SwarmDemo
```

### Capability showcase


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [christopherkarani/Swarm](https://github.com/christopherkarani/Swarm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->

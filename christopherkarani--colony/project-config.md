---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Test

Colony is a Swift Package (`swift-tools-version: 6.2`) targeting **iOS 26+** and **macOS 26+**.

**Dependency:** Colony builds on [Swarm](https://github.com/christopherkarani/Swarm) (orchestration primitives). Production builds resolve Swarm from GitHub at the version pinned in `Package.swift` (currently `0.5.0`). For local development against a sibling Swarm checkout at `../Swarm`, set `COLONY_USE_LOCAL_SWARM_PATH=1` before resolving. Note: `HIVE_DEPENDENCY.lock` and `scripts/ci/bootstrap-hive.sh` are legacy artifacts from the pre-Swarm era — the README still references them but the actual dependency is Swarm.

```bash
swift build                                      # build all targets
swift test                                       # run all test targets
swift test --filter ColonyTests                  # run only the core library tests
swift test --filter ColonyControlPlaneTests      # run only control plane tests
swift test --filter ColonyExecutionHardeningTests
swift test --filter ColonyResearchAssistantExampleTests
swift test --filter ColonyTests.ColonyAgentTests/colonyInterruptsAndResumesApproved  # single test
swift run ColonyResearchAssistantExample --help  # run the example CLI
```

## Architecture

### Module Split

| Target                    | Kind         | Purpose                                                                                                                 |
|---------------------------|--------------|-------------------------------------------------------------------------------------------------------------------------|
| `ColonyCore`              | library      | Pure value types, protocols, policies. Capabilities, configuration, tool approval, compaction, summarization, scratchbook, filesystem/shell/git/LSP/subagent backend contracts, built-in tool definitions, tokenizer, prompts. |
| `Colony`                  | library      | Runtime orchestration built on Swarm primitives. Agent graph (`ColonyAgent`), fluent builder (`ColonyBuilder`), runtime wrapper (`ColonyRuntime`), Foundation Models client, on-device/provider routers, MiniMax OpenAI client, durable checkpointing, observability, harness session. |
| `ColonyControlPlane`      | library      | Project/session management service. REST route descriptors, project store, session store, transport abstraction. Depends on `Colony` + `ColonyCore`. |
| `ColonyResearchAssistantExample` | executable | CLI research assistant example. |

`Colony` does `@_exported import ColonyCore`, so downstream consumers only need `import Colony`. Both libraries depend on `Swarm` (from the `Swarm` package product).

### Runtime Loop

The agent graph (`Sources/Colony/ColonyAgent.swift`) is compiled into a Swarm graph with these nodes:

```
preModel → model → routeAfterModel → tools → toolExecute → preModel
```

- `preModel` — patches dangling tool calls, runs summarization if policy is configured + filesystem is present, applies the compaction policy, writes `llmInputMessages`.
- `model` — builds the system prompt (memory, skills, scratchbook view, optional tool list), enforces `requestHardTokenLimit`, calls the model client.
- `routeAfterModel` — routes to `tools` if the model produced tool calls, otherwise terminates with a final answer.
- `tools` — selects/validates tool calls, raises approval interrupts when policy demands.
- `toolExecute` — dispatches through backends, writes results back, loops to `preModel`.

The loop terminates when the model produces a final answer (no tool calls) or the runtime yields an `.interrupted` outcome (e.g., tool-approval required).

### Capability-Gated Tool Families

Tools are only injected into the prompt/schema when the capability is enabled **and** the backend is wired. Capabilities live in `ColonyCore/ColonyCapabilities.swift` as an `OptionSet`.

| Capability        | Tools                                                                       | Backend Protocol                    |
|-------------------|-----------------------------------------------------------------------------|-------------------------------------|
| `.planning`       | `write_todos`, `read_todos`                                                 | (built-in)                          |
| `.filesystem`     | `ls`, `read_file`, `write_file`, `edit_file`, `glob`, `grep`                | `ColonyFileSystemBackend`           |
| `.shell`          | `execute`                                                                   | `ColonyShellBackend`                |
| `.shellSessions`  | `shell_open`, `shell_read`, `shell_write`, `shell_close`                    | `ColonyShellBackend` (session-capable) |
| `.scratchbook`    | `scratch_read/add/update/complete/pin/unpin`                                | (built-in, filesystem-backed)       |
| `.subagents`      | `task`                                                                      | `ColonySubagentRegistry`            |
| `.git`            | `git_status`, `git_diff`, `git_commit`, `git_branch`, `git_push`, `git_prepare_pr` | `ColonyGitService`           |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [christopherkarani/Colony](https://github.com/christopherkarani/Colony) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-22 -->

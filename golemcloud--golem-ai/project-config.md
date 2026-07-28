---
trigger: always_on
description: This is a **Golem Application** — a distributed computing project targeting WebAssembly (WASM). Components are compiled to `wasm32-wasip1` and executed on the Golem platform, which provides durable execution, persistent state, and agent-to-agent communication.
---

# Golem Application Development Guide (Rust)

## Overview

This is a **Golem Application** — a distributed computing project targeting WebAssembly (WASM). Components are compiled to `wasm32-wasip1` and executed on the Golem platform, which provides durable execution, persistent state, and agent-to-agent communication.

Key concepts:
- **Component**: A WASM module compiled from Rust, defining one or more agent types
- **Agent type**: A trait annotated with `#[agent_definition]`, defining the agent's API
- **Agent (worker)**: A running instance of an agent type, identified by constructor parameters, with persistent state

## Agent Fundamentals

- Every agent is uniquely identified by its **constructor parameter values** — two agents with the same parameters are the same agent
- Agents are **durable by default** — their state persists across invocations, failures, and restarts
- Invocations are processed **sequentially in a single thread** — no concurrency within a single agent, no need for locks
- Agents can **spawn other agents** and communicate with them via **RPC** (see Agent-to-Agent Communication)
- An agent is created implicitly on first invocation — no separate creation step needed

## Project Structure

```
golem.yaml                        # Root application manifest
Cargo.toml                        # Workspace Cargo.toml
components-rust/                  # Component crates (each becomes a WASM component)
  <component-name>/
    src/lib.rs                    # Agent definitions and implementations
    Cargo.toml                    # Must use crate-type = ["cdylib"]
    golem.yaml                    # Component-level manifest (templates, env, dependencies)
    .wit/wit/                     # WIT interface files (auto-managed)
common-rust/                      # Shared library crates (not compiled to WASM directly)
  common-lib/
    src/lib.rs
    Cargo.toml
  golem.yaml                     # Build templates for all Rust components
.wit/wit/deps/                   # Shared WIT dependencies (auto-managed)
golem-temp/                      # Build artifacts (gitignored)
```

## Prerequisites

- Rust with `wasm32-wasip1` target: `rustup target add wasm32-wasip1`
- `cargo-component` version 0.21.1: `cargo install --force cargo-component@0.21.1`
- Golem CLI (`golem`): download from https://github.com/golemcloud/golem/releases

## Building

```shell
golem build                      # Build all components
golem component build my:comp    # Build a specific component
golem build --build-profile release  # Build with release profile
```

The build compiles Rust to WASM, generates an agent wrapper, composes them, and links dependencies. Output goes to `golem-temp/`.

Do NOT run `cargo build` directly — always use `golem build` which orchestrates the full pipeline including WIT generation and WASM component linking.

## Deploying and Running

```shell
golem server run                 # Start local Golem server
golem deploy                     # Deploy all components to the configured server
golem deploy --try-update-agents # Deploy and update running agents
golem deploy --reset             # Deploy and delete all previously created agents
```

**WARNING**: `golem server run --clean` deletes all existing state (agents, data, deployed components). Never run it without explicitly asking the user for confirmation first.

After starting the server, components must be deployed with `golem deploy` before agents can be invoked. When iterating on code changes, use `golem deploy --reset` to delete all previously created agents — without this, existing agent instances continue running with the old component version. This is by design: Golem updates do not break existing running instances.

To try out agents after deploying, use `golem agent invoke` for individual method calls, or write a Rib script and run it with `golem repl` for interactive testing. The Golem server must be running in a separate process before invoking or testing agents.

## Name Mapping (Kebab-Case Convention)

All Rust identifiers are converted to **kebab-case** when used externally (in CLI commands, Rib scripts, REPL, agent IDs, and WAVE values). This applies to:

- **Agent type names**: `CounterAgent` → `counter-agent`
- **Method names**: `get_count` or `getCount` → `get-count`
- **Record field names**: `field_name` → `field-name`
- **Enum/variant case names**: `MyCase` → `my-case`

This conversion is automatic and consistent across all external interfaces.

## Testing Agents

### Using the REPL

```shell
golem repl                       # Interactive Rib scripting REPL
```

In the REPL, use kebab-case names and WAVE-encoded values:
```rib
let agent = counter-agent("my-counter")
agent.increment()
agent.increment()
```

### Using `golem agent invoke`

Invoke agent methods directly from the CLI. The method name must be fully qualified:

```shell
# Method name format: <component-name>/<agent-type>.{method-name}
# All names in kebab-case

golem agent invoke 'counter-agent("my-counter")' \
  'my:comp/counter-agent.{increment}'


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [golemcloud/golem-ai](https://github.com/golemcloud/golem-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->

---
trigger: always_on
description: `complier` is a Rust workspace for enforcing contracts over tool-using AI agents.
---

# AGENTS.md

## Purpose

`complier` is a Rust workspace for enforcing contracts over tool-using AI agents.

The core idea is simple:

- Developers already have agents running in existing frameworks.
- They should not need to replace those frameworks.
- They should be able to declare the process the agent is supposed to follow.
- `complier` should sit at the tool boundary and enforce that process.

When an agent attempts a tool call:

- if the call complies with the active contract, the tool runs
- if the call does not comply, the tool call is blocked
- the agent receives a structured remediation message explaining what happened and what it can do next

The product value is not "tool ordering" in isolation. The product value is:

- make your agent comply with the intended process

## What This Repo Is Building

This repo is building the enforcement layer as a Rust library plus MCP proxy binaries.

The current architecture is:

- `Contract`: the compiled runtime representation of the authored spec
- `Memory`: optional learned knowledge that can influence evaluation over time
- `Session`: one live execution against a contract and optional memory
- `FunctionWrapper`: wraps in-process async callables so they are enforced through a session
- `complier-mcp-proxy`: stdio MCP proxy that enforces a session against a downstream MCP server
- `complier-remote-mcp-proxy`: streamable-HTTP variant of the MCP proxy

There may be many workflows inside a single contract, so the primary top-level concept is `Contract`, not `Workflow`.

The authored source format is not the long-lived object in the system. The authored spec is parsed and compiled early, and the runtime operates on the compiled contract.

## Product Direction

This project should be treated like a real product, not a temporary prototype.

Important product assumptions:

- `complier` should work with existing agent frameworks rather than replacing them
- the library and binary ergonomics both matter — adoption paths are "link the crate" and "drop the proxy in front of my MCP server"
- the enforcement layer should feel general, not like a tiny niche feature
- the key abstraction is agent compliance with an intended process
- the system should govern what agents do, not just what they say

## Current Repo Shape

- `core/` — Rust workspace (this is the active implementation)
  - `core/ast/` — typed AST for the `.cpl` language
  - `core/parser/` — hand-written lexer and parser that produces an AST
  - `core/compiler/` — AST-to-runtime-graph compilation (`Contract`, `CompiledWorkflow`, `RuntimeNode`)
  - `core/runtime/` — shared node/graph types consumed by the session
  - `core/session/` — live execution state, decisions, memory, evaluators, remediation, session server client
  - `core/wrappers/` — `FunctionWrapper` plus the two MCP proxy binaries (`src/bin/mcp_proxy.rs`, `src/bin/remote_mcp_proxy.rs`)
- `archive/` — the Python prototype, preserved for reference only (do not extend it)
- `visualizer/` — Vite app that renders a compiled contract as a graph
- `landing/` — Next.js marketing site
- `assets/` — logo and other static assets

## Contract Syntax

The contract language is a DSL for declaring the process an agent is supposed to comply with.

At the top level, a contract contains:

- guarantees
- workflows

### Guarantees

Guarantees define reusable checks that can be referenced by workflows.

Example:

```text
guarantee safe [no_harmful_content]:halt
```

The current syntax supports checks such as:

- `[check]` for model-style checks
- `{check}` for human checks
- `#{check}` for learned human checks backed by memory

Checks may also include failure policies such as:

- `:halt`
- `:skip`
- `:3` for retries

Checks can be composed with boolean logic:

- `&&`
- `||`
- `!`

### Workflows

A contract may define many workflows.

Example:

```text
workflow "research" @always safe
    | @human "What topic?"
    | search_web
    | @llm "Summarize" ([relevant]:3 && [concise]:halt)
```

A workflow consists of:

- a name
- zero or more `@always` guarantees
- a series of pipe-prefixed steps

### Step Types

The DSL currently includes these major step forms:

- tool calls such as `search_web`
- tool calls with parameters such as `email to="user"`
- `@llm "Prompt"`
- `@human "Prompt"`
- `@call workflow_name`
- `@use workflow_name`
- `@inline workflow_name`
- `@branch`
- `@loop`
- `@unordered`
- `@fork id @call workflow_name`
- `@join id`

### Branches

Branches allow a workflow to choose between different paths.

Example:

```text
| @branch
    -when "technical"
        | @llm "Write detailed analysis"
    -when "general"
        | @llm "Write brief summary"
    -else
        | @llm "Write overview"
-end
```

### Loops

Loops repeat until a condition is satisfied.

Example:

```text
| @loop
    | @human "Is this good enough?"
    -until "yes"
-end
```

### Unordered Blocks

Unordered blocks represent a set of steps whose internal order does not matter.

Example:

```text
| @unordered
    -step format_citations
    -step generate_bibliography
-end
```

### Fork and Join

Fork and join allow parallel sub-work to be declared explicitly.

Example:

```text
| @fork refs @call check_references
| @fork refs @call verify_sources
| @join refs
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kavishsathia/complier](https://github.com/kavishsathia/complier) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->

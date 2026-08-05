---
trigger: always_on
description: This file is the operational guide for agents working in this repository. Keep
---

# Neoagent contributor guide

This file is the operational guide for agents working in this repository. Keep
it concise and update it whenever the development workflow or a hard invariant
changes.

## Product principles

- Keep it simple. Prefer a small explicit composition over a framework.
- Less is more. Do not add an abstraction until a concrete use case requires
  it.
- Neoagent's foundation is an LLM and agent API. Sessions, persistence,
  Workspace, bundled tools, configuration, and UI are optional higher-level
  compositions.
- Make every layer usable directly from ordinary Lua. Third-party plugins must
  be able to replace Models, tools, executors, message owners, and UI without
  patching global state.
- Prefer plain tables, functions, and constructors over registries, discovery,
  inheritance hierarchies, generic hook buses, or extension frameworks.
- Do not introduce built-in approval or permission policy. Approval prompts,
  logging, sandbox delegation, and similar policy belong in an
  `execute_tool(tool, arguments, ctx)` decorator.
- When a provider offers metered API and subscription or coding-plan access,
  support both compositions.
- Do not hardcode machine-specific paths. Executables and test dependencies
  must come from `PATH`, Make variables, environment variables, or
  repository-relative dependency directories.

## Architectural invariants

- `neoagent.api.*`, `neoagent.transport.*`, `neoagent.async`, and
  `neoagent.agent` form the reusable core. They must not import configuration,
  Sessions, storage, Workspace, bundled tools, the controller, or UI.
- A Model is an explicit value with `model:stream(opts)`. It uses named
  `on_event` and `on_done` options and returns a cancellable Run.
- `agent.run(opts)` receives its Model, messages, exact tools, executor, and
  context explicitly. It does not mutate input messages or resolve defaults.
- Steering enters the core through an explicit `get_steering_messages`
  callback and is consumed between assistant/tool turns. Each Controller owns
  its pending steering queue; the Window restores queued text for editing.
- `Session.new()` remains a no-argument, tool-free in-memory message owner. A
  store is optional and injected.
- The passive View consumes messages and events. A Window owns one View,
  selects an active Controller, and retains one input draft per Controller.
  Attached Controllers have unique, non-empty names.
- Controllers compose configuration, model selection, Session, Workspace, and
  Run. They publish transcript snapshots and updates while the Session retains
  the complete active branch.
- Controller Runs remain independent when a shared Window selects another
  Controller. The command-facing default Window is replaceable; custom
  Controllers and Windows must not mutate or depend on it.
- AGENTS.md and skill discovery are optional higher-level resource modules;
  reusable core layers do not depend on them.
- Bundled file tools operate only on disk. Loaded Neovim buffers are not a tool
  storage layer; the built-in Neo Controller may refresh an unmodified matching
  buffer after a successful disk mutation.
- `request_opts` is the sole built-in request customization mechanism. It may
  be a table or callback and recursively merges provider, model, then call
  layers across `url`, `headers`, and `body`.
- Thinking levels are model-declared request-option layers. The default
  controller selects and displays a level; Models and `agent.run()` do not
  interpret thinking semantics.
- Authentication wraps Models at stream time through injected login methods
  and credential storage. OAuth flows and Models remain independent from the
  command/UI adapter.
- The provider/model registry explicitly composes built-in defaults with user
  overrides without affecting direct Model constructors.
- Persist credentials atomically outside user configuration. Serialize login,
  refresh, and deletion; enumerate only secret-free credential metadata.
  Credential directories created by the store use mode `0700`; files use mode
  `0600`. Never log API keys, access tokens, or refresh tokens.
- Persistence remains compatible with the Pi v3 append-only tree format.
  Opening Neovim or creating an empty Session must not create a session file.
- Compaction receives its Session path and Model explicitly. Controllers own
  automatic compaction and overflow recovery.
- Provider diagnostics are bounded and never contain credentials, request or
  response bodies, or conversation content.
- Cancellation must propagate through active Models, tools, and nested Runs,
  complete exactly once, preserve meaningful partial output, and prevent stale
  callbacks from mutating newer controller state.
- Runtime code has no Lua plugin dependencies. Curl, `rg`, and `fd` are runtime
  executables; ImageMagick's `magick` is optional.

## Working in the repository

- Treat this repository as the canonical source. Do not edit or deploy a copied
  plugin installation unless the user explicitly asks for deployment.
- Write documentation and comments as a direct description of the current
  design. Do not preserve implementation history or discarded alternatives
  with phrases such as "not a ...", "rather than ...", "instead of ...",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tarruda/neoagent](https://github.com/tarruda/neoagent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->

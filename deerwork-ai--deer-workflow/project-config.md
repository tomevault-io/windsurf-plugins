---
trigger: always_on
description: `deer-workflow` is an open-source reimplementation of the Dynamic Workflow idea. The canonical repository is `https://github.com/deerwork-ai/deer-workflow`.
---

# deer-workflow

`deer-workflow` is an open-source reimplementation of the Dynamic Workflow idea. The canonical repository is `https://github.com/deerwork-ai/deer-workflow`.

The runtime keeps deterministic orchestration in TypeScript and delegates semantic work to replaceable Agent runtimes. The default Agent runtime is Codex CLI.

Do not copy private or proprietary implementations. Reproduce public behavior through clean-room interfaces, tests, and documentation.

## Runtime

- Use Bun for package management, scripts, tests, and process execution.
- Use strict TypeScript.
- Publish the package as `@deerwork-ai/deer-workflow` while keeping the CLI command
  named `deer-workflow`.
- Document `bun install --global @deerwork-ai/deer-workflow` as the primary global
  CLI installation. Use the GitHub installation only when explicitly
  describing an unreleased repository snapshot.
- Never describe bare `bun install` as a global installation; in this
  repository it installs local development dependencies and Git hooks.
- Keep `deer-workflow run` as the Workflow execution command.
- Keep `deer-workflow create` as the Agent-backed generator. It accepts a user
  prompt from arguments or stdin, explicitly directs Codex to the bundled
  `skills/workflow-creator/SKILL.md`, appends the user prompt, and writes only
  generated source to stdout.
- Resolve the bundled Skill relative to the installed CLI module so `create`
  works from a global GitHub or npm installation. Do not depend on the caller
  having installed `workflow-creator` in a Codex Skill search directory.
- Run the create Agent with a read-only sandbox and allow execution outside a
  Git repository. Strip one enclosing Markdown source fence before writing
  stdout so shell redirection produces a runnable source file. Before starting
  the Agent, write
  `/* Generating a DeerFlow Dynamic Workflow with Codex */` to stdout so a
  redirected target is immediately non-empty.
- Route CLI Workflow events to stderr as JSON Lines when stderr is redirected.
  In an interactive terminal, drive the run TUI from typed events instead.
  Keep the final result on stdout in both default modes. With `run --print` or
  `run -p`, disable the TUI, write one JSON event per stdout line, reserve
  stderr for CLI diagnostics, and suppress the separate final result. Present
  Print Mode as the recommended interface for servers and automation.
- Resolve optional run input in this order: `--input`, `--input-file`, then
  non-empty stdin. Reject simultaneous `--input` and `--input-file`; explicit
  options take precedence over stdin.
- Use the `tsconfig.json` path aliases to exercise public
  `@deerwork-ai/deer-workflow/*` imports locally.
- Keep runnable examples under `examples/<example-name>/`, with types in
  `types.ts`, the Workflow entry point in `workflow.ts`, and reciprocal English
  and Simplified Chinese README files.
- Link relevant examples from both language variants of the root README,
  Getting Started guide, and API reference.
- Keep the CLI entry point at `src/cli.ts`.
- Keep all Agent type aliases and interfaces in `src/agents/types.ts`.
- Keep the vendor-neutral Agent binder in `src/agents/agent.ts`.
- Keep Codex-specific process handling in `src/agents/codex-agent.ts`.
- Detect a missing Codex executable before creating temporary files or starting
  a process. The error must include official CLI installation steps and state
  that Codex CLI and Codex Desktop are separate installations.
- Re-export the default `agent()` function from `src/agents/index.ts`.
- Keep all Flow type aliases and interfaces in `src/flow/types.ts`.
- Keep deterministic orchestration primitives in `src/flow/`.
- Mirror flow tests under `tests/flow/`.
- Keep all Logging type aliases and interfaces in `src/logging/types.ts`.
- Keep Logging implementations in `src/logging/` and tests in
  `tests/logging/`.
- Keep all Workflow Event type aliases and interfaces in `src/events/types.ts`.
- Keep Event implementations in `src/events/` and tests in `tests/events/`.
- Keep all Runner type aliases and interfaces in `src/runner/types.ts`.
- Keep Runner implementations in `src/runner/` and tests in `tests/runner/`.
- Write `log()` messages directly to stderr when no Log Sink is active.
- Emit Runner events as JSON Lines. A standalone Runner's default `logWriter`
  calls `console.log` once per event, while the CLI sends redirected events to
  stderr and uses typed events for its interactive TUI so stdout remains
  reserved for the final result.
- Keep Workflow arguments and results out of events by default. Event payloads
  must remain JSON-safe and suitable for external process boundaries.
- Workflow modules export a handler as either `default` or `run`.
- Workflow Creator output also exports a pure-literal `meta` object with a
  kebab-case `name`, one-line `description`, and unique `phases` whose titles
  exactly match `phase()` calls, plus JSON-safe `exampleArgs` whose keys match
  Handler `args` properties. The Runner validates this export and emits
  `workflow:meta`; the interactive CLI uses its phase plan in the run TUI, and
  `create` uses the example args in its next command.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [deerwork-ai/deer-workflow](https://github.com/deerwork-ai/deer-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->

---
trigger: always_on
description: This is the operating model for how engineers design, implement, review, and evolve code in this repository.
---

# Modus Operandi

This is the operating model for how engineers design, implement, review, and evolve code in this repository.

## Key Directives

- If there was a design doc, ALWAYS re-read it after compaction.
- Always git clone dependencies into /tmp to inspect if necessary
- ALWAYS refer to Svelte 5, either docs or by cloning the repo, to make sure we're following best practices and canonical patterns
- DO NOT add to tech debt. It is CRITICAL that we keep the architecture clean and rational, even if that means taking longer to fix or refactor what we're working on.
- Use `bun` instead of `npm` or `npx`.
- Use `bun run start --port 0` and a timeout to validate that the server can compile and startup iff there have been code changes.
- DO NOT kill or pkill any running server processes. When testing, always start a NEW server on a different port (e.g. `--port 0` for random or a specific unused port). The user's primary server must never be disrupted.
- Run `bun run test` to validate your changes
- DO NOT use `sed`, remove items using tools if needed.
- DO NOT stop until the goal has been achieved
- DO NOT git commit or modify the git tree in any way, treat it as read-only.
- DO NOT run tests in the background and sleep for variable amounts of time to wait for them to complete, simply run them in the foreground instead.
- DO NOT run the same tests again and again to grep for different output. Instead, forward 2>&1 and `tee` the cargo test to a /tmp file, and grep from it after.
- DO NOT use emojis
- If interacting with the Claude, Codex, or Opencode SDK, clone it and look through if as needed:
  - https://github.com/anthropics/claude-agent-sdk-python
    - this is the Python SDK - the Typescript one is closed source, but you can find references in our node_modules
  - [https://github.com/openai](https://github.com/openai) - SDK contained inside
  - [https://github.com/anomalyco/opencode](https://github.com/anomalyco/opencode) - SDK contained inside

## Comment Style

- Always be concise
- Use third-person declarative form, eg. "Executes the provided command."
- Include comments that would be helpful for future changes and where the rationale isn't clear from the code. 
- DO NOT use separator lines or emojis, eg === or ---
- DO NOT enumerate steps, eg "N.", "Step N." or "Part N" - simply mention what is happening
- DO NOT include comments that are already clear from the code

## WebSocket and API Contract Discipline

- Protocol payloads must be typed on both sender and receiver paths.
- Message type names and fields must be stable and explicit.
- Add contract tests when introducing/changing payload shape.

Required for every WS/API contract change:

- Update type definitions.
- Update sender and receiver logic.
- Add or update tests.
- Add migration note in PR description if behavior changed.

## Tool Use Contract Discipline

- Normalize tool-use messages on the server side before they cross the shared boundary.
- Keep the client provider agnostic at the registry and renderer layers.
- Map generic cross-provider tools to shared canonical message types such as `bash-tool-use`, `read-tool-use`, `edit-tool-use`, `write-tool-use`, `web-search-tool-use`, and `web-fetch-tool-use`.
- When a provider emits a tool that cannot be represented cleanly as an existing generic tool-use message, add an explicit provider-specific shared message type instead of leaking the raw provider tool name into the client.
- Name provider-specific tool-use messages with an explicit provider prefix, for example `amp-oracle-tool-use`.
- Do not ship known tool behavior through `UnknownToolUseMessage`.
- Do not add or preserve client parsing or rendering paths that depend on `unknown-tool-use` for known tool families.
- Do not key frontend display behavior off `UnknownToolUseMessage.rawName`.
- Keep provider-specific translation logic inside `server/providers/converters/*`.
- Keep `common/chat-types.ts` as the single shared contract for all rendered tool-use messages, including provider-specific explicit variants.

Required for every known tool-use addition or change:

- Update `common/chat-types.ts` with the explicit message class, parser support, and union membership.
- Update the relevant provider converter to emit the explicit tool-use class instead of `UnknownToolUseMessage`.
- Update frontend display contracts and registry entries to resolve by message `type`, not provider raw name.
- Add or update converter tests, shared round-trip tests, and frontend rendering tests.
- Remove any client-side aliasing or raw-name rule that the new explicit type replaces.

## Clean Code Rules (Practical)

- Name by domain intent, not implementation detail.
- Keep functions small and single-purpose.
- Avoid boolean-flag overload APIs; prefer specific methods.
- Avoid duplicated business logic across components.
- Avoid "magic strings" crossing module boundaries without types/constants.
- Keep comments high-signal: explain why, not what.
- Remove dead paths quickly.

## Quality Gate

A task is not complete until:

- Scope and ownership are clear.
- New code follows Svelte 5 canonical patterns.
- Side effects are justified and cleaned up.
- Contracts are typed and tested.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cfal/garcon](https://github.com/cfal/garcon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->

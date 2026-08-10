---
trigger: always_on
description: This file contains repository-wide instructions for coding agents working on FigCC.
---

# AGENTS.md

This file contains repository-wide instructions for coding agents working on FigCC.

## Project identity

- Product name: **FigCC**.
- Compatibility package/service identifiers remain `figcodex`, `FIGCODEX_*`, and `com.figcodex.bridge`.
- FigCC is a derivative of [PavelLaptev/FigClaw](https://github.com/PavelLaptev/FigClaw), used under the MIT License.
- Preserve [LICENSE](LICENSE), [NOTICE.md](NOTICE.md), the upstream copyright line, and the explicit FigClaw attribution in both READMEs.
- This is an independent community project. Do not imply official affiliation with Figma, Anthropic, or OpenAI.

## Runtime architecture

```text
                                      ┌→ codex app-server
Figma iframe UI → authenticated bridge┤
       ↑ tool calls and results ↓      └→ Claude Code Agent SDK
Figma plugin sandbox → open Figma document
```

- `src/UI.svelte` owns the iframe, WebSocket client, provider-scoped conversation state, streamed events, and tool-call routing.
- `src/code.ts` owns Figma APIs, client storage, selection snapshots, downloads, and tool execution.
- `bridge/server.js` owns pairing authentication, provider dispatch, Codex threads, Claude sessions, attachments, skills, and tool routing.
- `bridge/codex-app-server.js` is the JSON-RPC stdio adapter.
- `bridge/claude-provider.js` is the Claude Code Agent SDK adapter and in-process FigCC MCP server.
- `bridge/dynamic-tool-reviewer.js` performs the independent fail-closed review for side-effecting FigCC tools.
- `bridge/skill-store.js` owns canonical filesystem skill packages under `skills/<name>/SKILL.md`.
- `bridge/workspace-store.js` owns the authenticated native folder selection and persisted project root; `bridge/linked-skill-store.js` overlays that project's live `skills/` packages on the bundled set.
- `src/tools.ts` is the dynamic-tool schema source of truth.
- Do not replace the App Server flow with one `codex exec` subprocess per prompt; that breaks the in-turn client tool handshake and native thread resume.
- A chat belongs to exactly one provider. Switching Codex/Claude starts a new empty chat; History restores the chat's recorded provider and only its native thread/session ID. Never import transcript text across providers.

## Security invariants

Do not weaken these without an explicit, security-reviewed request:

- The bridge binds to loopback by default and requires the pairing token.
- Pairing tokens, Codex/Claude credentials, `.figcodex-data/`, and `.figclaw-data/` must never enter git, prompts, UI diagnostics, or public logs.
- Codex defaults to the live `:read-only` permission profile with `approvalPolicy: on-request`, `approvalsReviewer: auto_review`, and this project as its narrow runtime root. Workspace and full-access profiles remain explicit user choices.
- Claude defaults to the locked `:read-only` profile, uses the local Claude Code login through the official Agent SDK, and never asks FigCC for an Anthropic API key. Workspace, Auto, and Full access remain explicit choices.
- Figma canvas inspection and mutations run directly through the plugin sandbox. Skill writes, downloads, and local project-file escalations retain their applicable review boundary.
- Bridge-side review for actions that still require it fails closed. A parse error, timeout, unavailable reviewer, or uncertain decision is not approval.
- Keep compatible user-configured MCP servers disabled for the dedicated canvas agent unless a deliberate architecture change is approved and tested.
- Keep the Figma manifest allowlist narrow. Never add wildcard network domains.
- Validate fetch targets in code in addition to the manifest.
- Selection metadata and previews stay local until the user presses Send.
- Bound node counts, text lengths, image counts, image sizes, and serialized tool results.

## Compatibility and storage

- New visible product copy uses FigCC. Existing `figcodex` technical identifiers remain stable for compatibility.
- Legacy FigClaw storage keys, `.figclaw-data`, pairing tokens, and environment variables are read only as migration fallbacks.
- Do not remove a migration fallback without documenting the breaking change.
- Preserve Codex thread IDs, Claude session IDs, each provider's policy version, and provider ownership when changing chat history.
- Model and reasoning-effort choices must come from each live provider catalog. Permission choices must reflect the installed provider's supported runtime modes; do not hard-code marketing model names.
- `skills/` remains the bundled canonical source. Keep `.agents/skills -> ../skills` and `.claude/skills -> ../skills`; do not fork provider-specific copies. A user-selected workspace may overlay live packages from its own `skills/` folder, with same-id workspace packages taking precedence.

## Pairing-token assistance

- When the plugin reports that the pairing token was rejected, first run `npm run bridge:status` and verify that `.figcodex-data/bridge-token` exists and is non-empty. Compare migration tokens only with a boolean check such as `cmp -s`; never print either value.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [madebypan/FigCC](https://github.com/madebypan/FigCC) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->

---
trigger: always_on
description: Canvasight is an early-stage repo-local Codex plugin. The active product lives under `plugins/canvasight` as a Vite, React, TypeScript, XYFlow, Zustand, and Radix UI application. Preserve room for the final stack, but do not treat the repository as an empty baseline anymore.
---

# AGENTS.md

## Project Context

Canvasight is an early-stage repo-local Codex plugin. The active product lives under `plugins/canvasight` as a Vite, React, TypeScript, XYFlow, Zustand, and Radix UI application. Preserve room for the final stack, but do not treat the repository as an empty baseline anymore.

The plugin opens a canvas workspace for arranging task nodes, attachments, and prompt flows. The web app is served by a project-level local daemon that outlives thread-local MCP shim processes. Normal plugin use renders Canvasight inside a Codex native widget through `open_canvasight`; callers must read the active task's `CODEX_THREAD_ID` and pass it as `threadId` because MCP child processes may not inherit it and Chat / Plan / Goal preflight requires an explicit target. The widget directly hosts the built app instead of iframe-loading localhost. Native widget JSON APIs route through the app-only `canvasight_widget_api` MCP tool, which proxies a strict allowlist to the daemon; the sandboxed widget must not depend on direct localhost fetches for startup or editing. Widget CSP must still include the daemon's exact origin before resource delivery for daemon-backed attachment assets. Native open tool results must not publicly expose daemon `127.0.0.1` URLs or tokens. Running a node sends Markdown for that node plus downstream children to the current Codex thread through the widget host bridge after native Chat / Plan / Goal preflight. The widget bridge may use standard MCP Apps `ui/message` or the Codex/OpenAI compatibility `window.openai.sendFollowUpMessage`; both count as native widget host bridge transports, not browser fallback. A successful MCP App handshake makes `sendMessage` callable even if the host omits the advisory `hostCapabilities.message` declaration; only the send Promise result determines sent/failed. Browser URLs and bare dev pages are fallback/development surfaces: after a current-thread claim they only queue payloads for `await_canvasight_run`; they must not report app-server `turn/start` as a successful Run path.

`open_canvasight` tool completion means only that the MCP call created a provisional `OpenAttempt`; it is not proof that the canvas opened. Callers must pass its `sessionId`, `openAttemptId`, and exact `threadId` to `await_canvasight_widget_ready`. Native opening succeeds only when that result is verified for a concrete fullscreen `widgetInstanceId` with React mounted, project hydration complete, and a visible non-zero canvas. Until that acknowledgement is observed, the result is `unverified`; agents must not say the canvas is "opened", "ready", or "fixed" based on tool output, another renderer, resource reads, daemon health, browser fallback, or synthetic harnesses alone.

Use `design.md` as the product and UI design baseline when adding user-facing screens.

## Working Rules

- Read the existing file tree before making changes.
- Keep changes scoped to the user's request.
- Do not rewrite or remove user changes unless explicitly asked.
- Prefer small, reviewable commits and focused files over broad refactors.
- Add dependencies only when they solve a concrete implementation need.
- When introducing a build system, document the commands in this file and in the project README if one exists.

## Agent Roles

- Main thread: owns integration, architecture decisions, conflict handling, verification, and final delivery.
- Product Agent: keeps the plugin aligned with the product goal of a browser canvas that returns output to Codex.
- Design Agent: protects the web UI direction, component language, visual density, and removal of old desktop-shell residue.
- Development Agent: owns implementation across MCP, persistence, React, and build/runtime behavior.
- Test Supervisor Agent: owns smoke tests, build checks, plugin validation, and browser-visible verification.
- Customer Support Agent: owns user-facing README documentation. On every user-visible feature, command, installation, workflow, or troubleshooting change, this agent must decide whether `README.md` needs an update. When it does, update README in the same delivery. README must keep a bilingual switch structure with Chinese and English sections, and must explain what Canvasight is for, its main features, basic usage, plugin setup, development commands, and common questions.
  Before updating README, check `AGENTS.md`, `design.md`, `plugins/canvasight/package.json`, `plugins/canvasight/mcp/server.mjs`, and all `plugins/canvasight/skills/*/SKILL.md` files so commands, tool names, and feature descriptions stay current. Do not present development-only commands as normal user workflow.
- Skill Expert Agent: owns Codex Skill trigger boundaries, frontmatter descriptions, `SKILL.md` concision, reference splitting, and skill validation. This role should review any change under `plugins/canvasight/skills/`. If a dedicated subagent cannot be created because of tool thread limits, the main thread must perform the role explicitly using the `skill-creator` guidance and record the limitation in `agent-reports/`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Niall-Young/Canvasight](https://github.com/Niall-Young/Canvasight) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-13 -->

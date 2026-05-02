---
trigger: always_on
description: Do not create or switch to a new branch unless the user explicitly asks for it. Default to staying on the current branch, including `main`, unless the user directs otherwise.
---

# Repository Guidelines

## Branching (CRITICAL)
Do not create or switch to a new branch unless the user explicitly asks for it. Default to staying on the current branch, including `main`, unless the user directs otherwise.

## Project Structure & Module Organization
`src/` holds the TypeScript app; `src/index.ts` bootstraps the runtime. Subsystems live in `src/runtime/`, `src/guardian/`, `src/tools/`, `src/channels/`, `src/llm/`, and `src/search/`. Keep tests beside the code they cover as `*.test.ts`. The web UI is in `web/public/`. Use `scripts/` for verification harnesses, `docs/` for architecture and specs, `policies/` for rule files, `skills/` for bundled skills, and `native/windows-helper/` for the Rust helper.

## Build, Test, and Development Commands
- `npm run dev`: start the app with `tsx src/index.ts`.
- `.\scripts\start-dev-windows.ps1 -StartOnly`: start the Windows dev server without rebuilding for browser/API regression loops.
- `bash scripts/start-dev-unix.sh --start-only`: start the Unix/WSL dev server without rebuilding.
- `npm run build`: compile TypeScript to `dist/`.
- `npm run check`: type check with `tsc --noEmit`.
- `npm test`: run the Vitest suite.
- `npm run test:coverage`: run Vitest with coverage thresholds.
- `npx vitest run src/path/file.test.ts`: run one test file.
- `npm run validate:dependency-contract`: validate pinned dependency and lockfile contract after dependency or override changes.
- `npm run validate:windows-package`: validate staged Windows package manifests after Windows packaging work.
- `npm run helper:windows`: rebuild the Windows native helper when touching `native/windows-helper/`.
- **Integration Testing (CRITICAL):** Review `docs/guides/INTEGRATION-TEST-HARNESS.md` for full-stack API regression testing, including guidance for cross-platform (Windows/WSL) and Ollama configurations.

## Intent Gateway (CRITICAL)
All user intent classification must go through the Intent Gateway (`src/runtime/intent-gateway.ts`). **Never use regex, keyword matching, string includes, or any ad-hoc pattern matching to determine what the user is asking for.** The Intent Gateway is an LLM classifier that routes requests via structured tool calls. New routes must be added to `IntentGatewayRoute`, the tool schema, the system prompt, `normalizeRoute`, `preferredCandidatesForDecision` in `direct-intent-routing.ts`, and the candidate dispatch loop in `src/index.ts`. Pre-gateway interception is only permitted for slash-command parsing in channel adapters and continuation/approval flow detection.

## Shared Orchestration (CRITICAL)
When a bug is about blocked execution, prerequisites, approvals, clarifications, cross-turn resume, workspace switching, or channel-specific drift, fix it by extending the shared orchestration/state system first. In the current architecture that means the Intent Gateway contract, `PendingActionStore`, shared response metadata, and shared channel rendering. Do not add bespoke per-tool or per-capability resume flows unless the shared model cannot represent the behavior.

## Architecture Discipline (CRITICAL)
Do not ship tactical workarounds that bypass the intended architecture just to make a failing path appear to work. Fix the root cause in the layer that owns the behavior:
- intent/routing bugs: `IntentGateway` and shared routing/orchestration
- blocked-work / approval drift: shared pending-action and channel metadata flow
- config/provider mutation: control-plane services and transactional config update paths
- tool visibility/discovery: the deferred-loading / `find_tools` design, unless there is an intentional architecture change

If a proposed fix would bypass the documented design, stop and reconsider. Examples of fixes that are not acceptable by default:
- promoting deferred tools to always-loaded just because a model failed to call `find_tools`
- adding one-off channel behavior that duplicates shared orchestration
- bypassing control-plane callbacks with ad hoc config writes
- adding special-case routing logic before the Intent Gateway

If the right fix is to change the architecture, make that an explicit architectural change and update the relevant docs/specs in the same change rather than sneaking in a workaround. For tool-loading changes, read `docs/design/TOOLS-CONTROL-PLANE-DESIGN.md`. For module boundaries and ownership, read `docs/architecture/FORWARD-ARCHITECTURE.md`.

## Strategic Architecture Pause (CRITICAL)
When a fix starts adding layers of adapters, compatibility shims, prompt rules, duplicated state, or per-channel/per-tool exceptions, pause before continuing. Re-read the owning design docs and current implementation, then decide whether the better move is an architecture uplift instead of another local patch.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Threat-Vector-Security/guardian-agent](https://github.com/Threat-Vector-Security/guardian-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->

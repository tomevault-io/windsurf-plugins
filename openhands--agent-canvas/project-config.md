---
trigger: always_on
description: - This repository is a near-direct port of the OpenHands frontend, adapted to talk straight to `software-agent-sdk` / `agent_server` without the usual OpenHands app backend.
---

# Repository Notes

## General

- This repository is a near-direct port of the OpenHands frontend, adapted to talk straight to `software-agent-sdk` / `agent_server` without the usual OpenHands app backend.
- Frontend API adaptation lives mainly in `src/api/`:
  - `option-service` fabricates an OSS web-client config and reads models/providers through `@openhands/typescript-client` LLM endpoints.
  - `settings-service` uses `@openhands/typescript-client` settings APIs for persistence; reads schemas from `/api/settings/agent-schema` and `/api/settings/conversation-schema`, fetches settings with optional `X-Expose-Secrets: encrypted` header for conversation start payloads, and saves settings via PATCH with diffs.
  - `agent-server-conversation-service`, `event-service`, `agent-server-git-service`, and `skills-service` route local agent-server access through `@openhands/typescript-client` rather than direct HTTP calls.
- Supported env vars for deployment:
  - `VITE_BACKEND_BASE_URL` for the agent server base URL.
  - `VITE_SESSION_API_KEY` for optional session auth.
  - `VITE_WORKING_DIR` for the default workspace path sent when starting conversations.
  - `VITE_ENABLE_BROWSER_TOOLS=false` to omit `BrowserToolSet` from new conversation payloads.
- Public skills are loaded from the `@openhands/extensions` npm package at build time via `SKILLS_CATALOG` (exported from `@openhands/extensions/skills`). The frontend's `SkillsService` maps catalog entries to `SkillInfo` objects and merges them with user/project skills fetched from the agent-server (with `load_public: false`). The agent-server no longer clones the extensions repo or uses `EXTENSIONS_REF` for public skills.
- Default working-dir fallback is now the relative path `workspace/project` (exported as `DEFAULT_WORKING_DIR` from `src/api/agent-server-config.ts`); git-path heuristics and the default PLAN preview path should reuse that constant instead of hardcoding `/workspace/project`.
- The UI keeps most OpenHands routes/layout intact, but hosted-only behavior (org, account management, integrations) has been removed via the fabricated OSS config because there is no separate app backend.
- Verification command: `npm run typecheck && npm run build`.
- GitHub automation now includes `.github/workflows/ci.yml` for `npm ci`, `npm test`, and `npm run build`, plus `.github/dependabot.yml` with weekly npm/github-actions updates gated by a 7-day cooldown.

## PR Description Human Check

The `HUMAN:` section and the `A human has tested these changes.` checkbox in
PR descriptions are reserved for human contributors only. AI agents
MUST NOT add to, edit, move, remove, or check these fields. If the PR description
CI fails because these fields are missing, empty, or unchecked, stop and ask the
human user to update them in their own words. If the fields were already updated
by a human, report the exact validator error rather than editing them yourself.

## Tracking / Analytics Architecture

Two distinct PostHog systems exist. **Never mix them at a call site.**

### System 1 — `telemetry.ts` (library-level, anonymous)
- **Purpose**: anonymous npm-consumer telemetry (`canvas_install`, `canvas_new_session`)
- **Keys**: hardcoded staging/prod keys in `telemetry.ts`; routed through `https://z.openhands.dev`
- **Consent**: `localStorage["openhands-telemetry-consent"]` via `useTelemetry` / `TelemetryConsentBanner`
- **`canvas_install`** fires once, pre-consent, per installation
- **Exports**: `trackEvent`, `useTelemetry`, `TelemetryConsentBanner`, etc. from `src/lib/index.ts` — these are the **public library API for npm consumers**
- **Rule**: Do NOT import `trackEvent` from `#/services/telemetry` in app routes or components

### System 2 — `useTracking` hook (app-level, identified)
- **Purpose**: product analytics for app behaviour events
- **Key**: `VITE_POSTHOG_CLIENT_KEY` env var → `OptionService.getConfig()` → `PostHogWrapper` → `PostHogProvider`; routed to `https://us.i.posthog.com`
- **Consent**: `user_consents_to_analytics` (backend setting) + `useSyncPostHogConsent` in `root-layout`; `AnalyticsConsentFormModal` also calls `setTelemetryConsent` to keep both systems in sync
- **All events** are typed, named functions in `src/hooks/use-tracking.ts` — add a new function there for every new event; never call `posthog.capture()` raw from a component
- **`commonProperties`** (`current_url`, `user_email`) are attached automatically by the hook
- **Rule**: Do NOT use raw `usePostHog()` + `posthog.capture()` in components — always go through `useTracking`

### Adding a new event
1. Add a typed function to `useTracking` in `src/hooks/use-tracking.ts`
2. Add the function to the hook's `return` object
3. Destructure and call it from the component: `const { trackFoo } = useTracking()`

### Env var
`VITE_POSTHOG_CLIENT_KEY` — see `.env.sample`. Without it, `PostHogProvider` never mounts and all `useTracking` calls are silently dropped (safe default for local dev).

## Runtime Services in Dev Stacks


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OpenHands/agent-canvas](https://github.com/OpenHands/agent-canvas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->

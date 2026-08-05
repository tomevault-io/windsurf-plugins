---
trigger: always_on
description: See [`.agents/retro.md`](.agents/retro.md) for lessons from past tasks. Read entries related to your current work before starting — they document mistakes that cost hours and how to avoid them.
---

# Mobile App - AGENTS.md

## Retrospectives (read before related work)

See [`.agents/retro.md`](.agents/retro.md) for lessons from past tasks. Read entries related to your current work before starting — they document mistakes that cost hours and how to avoid them.

## Agent Operating Rules (read first)

**Failure taxonomy — classify every failure before acting. Escalate cheapest-first; never skip a rung or repeat one:**
- **transient** (network blip, flaky): retry at most 2×.
- **code** (your bug, wrong path, bad arg): fix, then retry.
- **tool-usage-gated** (you called a *working* tool wrong): fix the *call*, not the tool. Cheapest fix — try this before assuming the tool is broken. Example: chrome-devtools `new_page` with `isolatedContext` opens a cookieless context; instead use `list_pages` → `select_page` to drive the existing authenticated tab.
- **tooling-gated** (the tool's *source* is genuinely broken): **diagnose before fixing** — verify it's actually broken (read the code, reproduce). If broken, spawn a fix-it subagent to patch + rebuild it, then retry. Do NOT stop.
- **infra** (CI runner, emulator boot timeout): report it; do not try to "fix" someone else's infrastructure.
- **human-gated** (only when no automated path exists): emit ONE precise instruction for the human, mark the task **BLOCKED**, and continue all other automatable work. Never re-attempt across turns.

**Diagnose before fixing:** Before patching or replacing any tool, confirm it is actually the cause. Copilot burned 39h assuming chrome-devtools-mcp was broken — it wasn't (it already drives the authenticated default context via `browser.defaultBrowserContext()`/`browser.pages()`). A 3-minute diagnostic subagent would have caught this.

**Known gated steps:**
- Play Console **API-access grant** to the service account — *was* the blocker; now granted (publish run 26662900471 succeeded, AAB on internal track, versionCode 19). One-time path if re-needed: Play Console → Setup → API access → link GCP project `opencode-mobile-deploy` → grant `playstore-deploy@opencode-mobile-deploy.iam.gserviceaccount.com` Admin/Release-manager.
- **Add internal testers** (Play Console → Internal testing → Testers): the genuinely human-gated residual. Google's anti-automation blocks CDP-controlled Chrome sign-in, so this UI step needs a human (or a real, non-CDP browser session).

**Stop discipline:** If a tool returns the same error 3× (or no new artifact/commit is produced across several turns), STOP. Print a BLOCKED summary with the single human action needed. A vague "please do X and let me know" that leaves you idle is worse than a clean stop — don't do it.

**Work-tracking discipline:**
- Track multi-step/upgrade work in the **related GitHub issue**, updated via `gh issue comment` — NOT by repeatedly editing AGENTS.md and NOT in scratch files under `/tmp` (e.g. no `/tmp/playconsole-fill.md`). Keep reference material (listing copy, form answers) in the repo under `distribution/` or as issue comments.
- AGENTS.md is for durable conventions only; do not churn it with task status.
- **Never claim a step done without verifying it in the real channel** (e.g. an app exists only if it appears in the Play Console app-list; an AAB is the right package only if its manifest says so). Do not invent IDs.
- **Driving web UIs:** snapshot → act on the *current* uids → re-snapshot. Never fire batched/guessed clicks; if a page shows "Loading"/an error toast, wait and re-snapshot rather than clicking blind.

## Overview

React Native / Expo mobile client for opencode. Connects to an opencode server instance via HTTP + SSE for real-time updates.

**Repo**: `dzianisv/opencode-mobile` (standalone, not part of opencode monorepo)
**Package name**: `cc.agentlabs.opencode`

## Architecture

```
app/                    # Expo Router file-based routing
├── (tabs)/             # Tab navigation (sessions, connections, settings)
├── session/[id].tsx    # Chat screen
└── connection/         # Add/edit connection screens
src/
├── components/         # Reusable UI components
│   ├── markdown/       # Markdown renderer (wraps react-native-marked)
│   └── AuthGate.tsx    # Biometric auth gate
├── lib/
│   ├── sdk.ts          # HTTP + SSE client for opencode server API
│   └── types.ts        # Re-exported types
└── stores/             # Zustand state stores
    ├── sessions.ts     # Session list, messages, parts
    ├── connections.ts  # Server connections, client lifecycle
    ├── events.ts       # SSE event stream, status tracking, permissions, questions
    └── auth.ts         # Biometric auth
scripts/
└── android-cua-smoke.py  # LLM-powered CUA E2E test
```

## Key Patterns

- **SSE for real-time**: The `events.ts` store connects to `/global/event` and dispatches to other stores
- **Fire-and-forget sends**: `sendMessage` posts to the API but doesn't await response; SSE events drive all UI updates
- **Session status**: Derived from `session.status` events (`idle`/`busy`/`retry`) + last part type for status text

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dzianisv/opencode-mobile](https://github.com/dzianisv/opencode-mobile) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->

---
trigger: always_on
description: Use when working on Slack agent/bot code, Chat SDK applications, Bolt for JavaScript projects, or projects using @chat-adapter/slack or @slack/bolt. Provides development patterns, testing requirements, and quality standards.
---


# Slack Agent Development Skill

This skill supports two frameworks for building Slack agents:

- **Chat SDK** (Recommended for new projects) — `chat` + `@chat-adapter/slack`
- **Bolt for JavaScript** (For existing Bolt projects) — `@slack/bolt` + `@vercel/slack-bolt`

## Skill Invocation Handling

When this skill is invoked via `/slack-agent`, check for arguments and route accordingly:

### Command Arguments

| Argument | Action |
|----------|--------|
| `new` | **Run the setup wizard from Phase 1.** Read `./wizard/1-project-setup.md` and guide the user through creating a new Slack agent. |
| `configure` | Start wizard at Phase 2 or 3 for existing projects |
| `deploy` | Start wizard at Phase 5 for production deployment |
| `test` | Start wizard at Phase 6 to set up testing |
| (no argument) | Auto-detect based on project state (see below) |

### Auto-Detection (No Argument)

If invoked without arguments, detect the project state and route appropriately:

1. **No `package.json` with `chat` or `@slack/bolt`** → Treat as `new`, start Phase 1
2. **Has project but no customized `manifest.json`** → Start Phase 2
3. **Has project but no `.env` file** → Start Phase 3
4. **Has `.env` but not tested** → Start Phase 4
5. **Tested but not deployed** → Start Phase 5
6. **Otherwise** → Provide general assistance using this skill's patterns

### Framework Detection

Detect which framework the project uses:

- **`package.json` contains `"chat"`** → Chat SDK project
- **`package.json` contains `"@slack/bolt"`** → Bolt project
- **Neither detected** → New project, recommend Chat SDK (offer Bolt as alternative)

Store the detected framework and use it to show the correct patterns throughout the wizard and development guidance.

### Wizard Phases

The wizard is located in `./wizard/` with these phases:
- `1-project-setup.md` - Understand purpose, choose framework, generate custom implementation plan
- `1b-approve-plan.md` - Present plan for user approval before scaffolding
- `2-create-slack-app.md` - Customize manifest, create app in Slack
- `3-configure-environment.md` - Set up .env with credentials
- `4-test-locally.md` - Dev server + ngrok tunnel
- `5-deploy-production.md` - Vercel deployment
- `6-setup-testing.md` - Vitest configuration

**IMPORTANT:** For `new` projects, you MUST:
1. Read `./wizard/1-project-setup.md` first
2. Ask the user what kind of agent they want to build
3. Offer framework choice (Chat SDK recommended, Bolt as alternative)
4. Generate a custom implementation plan using `./reference/agent-archetypes.md`
5. Present the plan for approval (Phase 1b) BEFORE scaffolding the project
6. Only proceed to scaffold after the plan is approved

---

## Framework Selection Guide

| Aspect | Chat SDK | Bolt for JavaScript |
|--------|----------|---------------------|
| **Best for** | New projects | Existing Bolt codebases |
| **Packages** | `chat`, `@chat-adapter/slack`, `@chat-adapter/state-redis` | `@slack/bolt`, `@vercel/slack-bolt` |
| **Server** | Next.js App Router | Nitro (H3-based) |
| **Event handling** | `bot.onNewMention()`, `bot.onSubscribedMessage()` | `app.event()`, `app.command()`, `app.message()` |
| **Webhook route** | `app/api/webhooks/[platform]/route.ts` | `server/api/slack/events.post.ts` |
| **Message posting** | `thread.post("text")` / `thread.post(<Card>...)` | `client.chat.postMessage({ channel, text, blocks })` |
| **UI components** | JSX: `<Card>`, `<Button>`, `<Actions>` | Raw Block Kit JSON |
| **State** | `@chat-adapter/state-redis` / `thread.state` | Manual / Vercel Workflow |
| **Config** | `new Chat({ adapters: { slack } })` | `new App({ token, signingSecret, receiver })` |

---

## General Development Guidance

You are working on a Slack agent project. Follow these mandatory practices for all code changes.

## Project Stack

### If using Chat SDK

- **Framework**: Next.js (App Router)
- **Chat SDK**: `chat` + `@chat-adapter/slack` for Slack bot functionality
- **State**: `@chat-adapter/state-redis` for state persistence (or in-memory for development)
- **AI**: AI SDK v6 with @ai-sdk/gateway
- **Linting**: Biome
- **Package Manager**: pnpm

```json
{
  "dependencies": {
    "ai": "^6.0.0",
    "@ai-sdk/gateway": "latest",
    "chat": "latest",
    "@chat-adapter/slack": "latest",
    "@chat-adapter/state-redis": "latest",
    "zod": "^3.x",
    "next": "^15.x"
  }
}
```

### If using Bolt for JavaScript

- **Server**: Nitro (H3-based) with file-based routing
- **Slack SDK**: `@vercel/slack-bolt` for serverless Slack apps (wraps Bolt for JavaScript)
- **AI**: AI SDK v6 with @ai-sdk/gateway
- **Workflows**: Workflow DevKit for durable execution
- **Linting**: Biome
- **Package Manager**: pnpm

```json
{
  "dependencies": {
    "ai": "^6.0.0",
    "@ai-sdk/gateway": "latest",
    "@slack/bolt": "^4.x",
    "@vercel/slack-bolt": "^1.0.2",
    "zod": "^3.x"
  }
}
```

**Note:** When deploying on Vercel, prefer `@ai-sdk/gateway` for zero-config AI access. Use direct provider SDKs (`@ai-sdk/openai`, `@ai-sdk/anthropic`, etc.) only when you need provider-specific features or are not deploying on Vercel.

---


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vercel-labs/slack-agent-skill](https://github.com/vercel-labs/slack-agent-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

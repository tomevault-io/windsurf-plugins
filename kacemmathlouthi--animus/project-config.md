---
trigger: always_on
description: Guidance for working in this repository. Keep this file current — it is the
---

# CLAUDE.md

Guidance for working in this repository. Keep this file current — it is the
single source of truth for what animus is and how we build it.

---

## What animus is

An **interactive coding agent for Manim videos** — Claude Code, but in the cloud
and specialized for narrated, research-grounded explainer animations. The user
chats with the agent; the agent writes and edits Manim code in a persistent
cloud sandbox, renders it, reads the errors / inspects frames, and repairs until
the scene is clean. The user can interrupt and steer at any turn (chat-only
steering — the agent owns the code). Narration uses `manim-voiceover` with
ElevenLabs, synthesized in-scene during render so the animation auto-syncs to the
speech; background music is mixed under it after rendering.

The mental model is a relocated Claude Code:

| Claude Code (local)        | animus (cloud)                                        |
| -------------------------- | ----------------------------------------------------- |
| Loop runs in your terminal | Loop runs in `apps/api`                               |
| Tools act on local files   | Tools act on a remote Daytona sandbox                 |
| Streams to the terminal    | Streams to the browser (SSE → `useChat` + Streamdown) |
| Ctrl-C to interrupt        | Abort the stream from the UI                          |
| General coding             | Manim tools + prompting, render-and-watch             |

The core insight: the **turn is ephemeral** (if it crashes or is interrupted,
the user just sends another message), and the **sandbox is the persistent
state** (the Manim project files live across turns).

---

## Monorepo layout

```
apps/
  web/    # React 19 + Vite SPA — landing, auth, studio (chat + conversation sidebar), settings, legal, /v/:token share page. plugins/ holds the dev link-preview meta-injection plugin (see the meta-injection seam decision).
  api/    # Hono/Bun — auth, settings, conversations, the streaming agent loop (/api/chat), and the public share endpoints (share-card og.png, video.mp4, embed). Long-running.
packages/
  core/   # shared contracts: zod schemas, types, constants + the deterministic share-card SVG builder + link-preview meta builders (pure root) + server env (/env subpath) + tool I/O contracts shared by the agent and the web (/tools subpath)
  auth/   # Better Auth (magic link via Resend, GitHub + Google OAuth, sessions)
  db/     # Drizzle + Postgres (schema, client, migrations)
  agent/  # the AI SDK agent loop, prompts, and the full tool set: HITL (askUserQuestion, finalizeVideoPlan), Exa web research (webSearch/webFetch), and the Manim sandbox tools (writeFile/editFile/runCommand/readFile/listFiles/renderScene). Ships the manim-video skill (skills/manim-video) and the prebaked snapshot Dockerfile.
```

Packages are **source-first**: no per-package build step. `apps/api` and the
packages run `.ts` directly via Bun; `apps/web` is the only thing Vite builds.
Cross-package "does it compile" = `bun run typecheck`.

---

## Architecture (locked decisions)

- **Interactive agent loop, not a batch pipeline.** Each user turn is an AI SDK
  tool-calling loop (`streamText` + Manim tools + `stopWhen`), streamed over SSE
  and interruptible. Durable execution (Vercel Workflow SDK) is **parked** for a
  future non-interactive "autonomous render" mode — it fights interactive streams.
- **Sandbox: Daytona**, one per conversation, suspended between turns and resumed.
  Sandboxes boot from a **prebaked snapshot** with Manim + ffmpeg + LaTeX already
  installed (no per-turn bootstrap). The snapshot is built from
  `packages/agent/snapshot/Dockerfile` via `bun run snapshot:build` (registers it
  as `animus-manim:<tag>`; the tag currently in use is `SNAPSHOT_NAME` in
  `packages/agent/src/sandbox/index.ts` — `animus-manim:0.9.1`). The lifecycle lives
  in `packages/agent/src/sandbox` as plain functions
  (`ensureSandbox`/`destroySandbox`) returning the Daytona handle directly — the
  tools call the SDK natively (no wrapper adapter).
- **Models: Claude via Amazon Bedrock (default), with BYOK.** By default the
  agent resolves its model through `@ai-sdk/amazon-bedrock` — a Bedrock
  inference-profile id (`BEDROCK_MODEL`, e.g. `us.anthropic.claude-opus-4-6-v1`).
  The **code default** is Opus 4.6; **prod is expected to set `BEDROCK_MODEL`
  explicitly** as an SSM parameter under `/animus/prod/`. Recommended prod value once
  Sonnet 5 model access is enabled in the Bedrock console:
  `us.anthropic.claude-sonnet-5` (~60% cheaper than Opus; the `@animus/core`
  pricing table already prices `claude-sonnet-5`, and `priceForModel`
  substring-matches the profile id, so metering stays correct). Leaving the env
  unset means prod silently runs the Opus default at Opus rates.
  Credentials are passed to the provider **explicitly** from the validated
  server env (`BEDROCK_ACCESS_KEY_ID`/`BEDROCK_SECRET_ACCESS_KEY`/
  `BEDROCK_REGION`, falling back to `AWS_*` so local `.env` files keep
  working): **never rely on the SDK's implicit `AWS_*` env chain in prod**. The
  `BEDROCK_*` names exist because Vercel owned the `AWS_*` namespace at runtime
  and shadowed user-supplied values, which silently broke every chat turn once;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KacemMathlouthi/animus](https://github.com/KacemMathlouthi/animus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-05 -->

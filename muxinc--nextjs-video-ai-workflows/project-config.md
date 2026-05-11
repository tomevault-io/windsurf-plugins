---
trigger: always_on
description: Guidance for AI coding assistants working on this project.
---

# AGENTS.md

Guidance for AI coding assistants working on this project.

---

## What this project is

A **reference architecture** demonstrating how to integrate `@mux/ai` with **Vercel Workflows** to ship video intelligence that holds up at scale.

The app ("Demuxed Library") uses real Mux assets to teach three integration layers:

| Layer | Pattern    | Example                                                       |
| ----- | ---------- | ------------------------------------------------------------- |
| **1** | Primitives | `getSummaryAndTags()` — call primitives directly              |
| **2** | Workflows  | `translateCaptions`, `translateAudio` — run workflows durably |
| **3** | Connectors | Clip creation — compose with external tools like Remotion     |

**Read the full context:**

- `context/application-explained.md` — what the app does and why
- `context/design-explained.md` — visual design and UX patterns
- `context/implementation-explained.md` — routes, data model, and code patterns

---

## Code style (ESLint)

This project uses `@antfu/eslint-config` with custom rules dictated within `eslint.config.mjs`. Key points:

### Formatting

- **Indent**: 2 spaces
- **Semicolons**: always
- **Quotes**: double quotes (`"`)
- **Brace style**: cuddled (`} else {` on same line)
- **Operators**: at end of line, not beginning

### Import ordering

Imports are sorted by the `perfectionist/sort-imports` rule:

```typescript
// 1. Side-effect styles
import "./styles.css";

// 2. Built-in modules
import { Buffer } from "node:buffer";

// 5. Parent/sibling/index
import { env } from "@/lib/env";
// 3. External packages
import { z } from "zod";

// 4. Internal (@mux/ai is treated as internal)
import { getSummaryAndTags } from "@mux/ai/workflows";
```

**Blank lines between groups are required.**

### File naming

- **kebab-case** for all files (e.g., `translate-captions.ts`, not `translateCaptions.ts`)
- Exception: `README.md` and other all-caps markdown files

### Console usage

- `console.log` triggers a warning — prefer structured logging or remove before committing

---

## Environment variables

All env vars are validated at startup via `app/lib/env.ts` using Zod.

### Required variables

```bash
# Mux credentials
MUX_TOKEN_ID=
MUX_TOKEN_SECRET=

# OpenAI (required for embeddings)
OPENAI_API_KEY=

# ElevenLabs (for translateAudio)
ELEVENLABS_API_KEY=

# S3-compatible storage (for translation workflows)
S3_ENDPOINT=
S3_REGION=
S3_BUCKET=
S3_ACCESS_KEY_ID=
S3_SECRET_ACCESS_KEY=
```

### Optional variables

```bash
# Mux signing keys (for signed playback URLs)
MUX_SIGNING_KEY=
MUX_PRIVATE_KEY=

# Additional AI providers
ANTHROPIC_API_KEY=
GOOGLE_GENERATIVE_AI_API_KEY=
```

### Accessing env vars

**Never use `process.env` directly.** Import from the validated env module:

```typescript
import { env } from "@/lib/env";

// ✅ Correct
const tokenId = env.MUX_TOKEN_ID;

// ❌ Wrong — bypasses validation, triggers ESLint error
// eslint-disable-next-line node/no-process-env
const tokenId = process.env.MUX_TOKEN_ID;
```

The ESLint rule `node/no-process-env` enforces this.

---

## Mux client (`app/lib/mux.ts`)

The shared Mux client and helpers live in `app/lib/mux.ts`. This module:

- Initializes a singleton `Mux` client from `@mux/mux-node`
- Exports typed helpers for asset retrieval, playback ID extraction, and track lookups
- Centralizes all credential access (uses validated `env` module)

**Always import from this module** rather than creating new `Mux` instances:

```typescript
import { getAsset, getPlaybackIdForAsset, listAssets } from "@/lib/mux";

// ✅ Correct — uses shared client
const asset = await getAsset(assetId);

// ❌ Wrong — creates duplicate client, bypasses centralized setup
const mux = new Mux({ tokenId: env.MUX_TOKEN_ID, tokenSecret: env.MUX_TOKEN_SECRET });
```

---

## Vercel Workflow patterns

### Directive placement

Per [Vercel Workflow docs](https://useworkflow.dev/docs/getting-started/next):

- `"use workflow"` goes **inside** the workflow function (first line)
- `"use step"` goes **inside** each step function (first line)

```typescript
// ✅ Correct
export async function myWorkflow(input: Input) {
  "use workflow";
  // orchestration logic
}

async function myStep(data: Data) {
  "use step";
  // business logic
}
```

### Starting workflows from route handlers

Per [Vercel Workflow docs](https://useworkflow.dev/docs/getting-started/next#create-your-route-handler), workflows are triggered via `start()` from `workflow/api` in a route handler:

```typescript
import { NextResponse } from "next/server";
import { start } from "workflow/api";

// app/api/workflows/translate-captions/route.ts
import { translateCaptionsWorkflow } from "@/workflows/translate-captions";

export async function POST(request: Request) {
  const { assetId, targetLang } = await request.json();

  // Executes asynchronously and doesn't block your app
  await start(translateCaptionsWorkflow, [assetId, targetLang]);

  return NextResponse.json({ message: "Workflow started" });
}
```

Key points:

- `start()` returns immediately — the workflow runs in the background
- Pass workflow arguments as an array (second argument to `start`)
- Workflows can be triggered from route handlers, server actions, or any server-side code


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [muxinc/nextjs-video-ai-workflows](https://github.com/muxinc/nextjs-video-ai-workflows) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->

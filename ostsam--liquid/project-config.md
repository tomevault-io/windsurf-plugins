---
trigger: always_on
description: **"The AI shouldn't just generate the answer; it should generate the controls to manipulate that answer."**
---

# Liquid Control — CLAUDE.md

## Project Vision

**"The AI shouldn't just generate the answer; it should generate the controls to manipulate that answer."**

Liquid is a Just-In-Time interface engine. The user pastes any content — an email, a code snippet, a tweet — and Claude analyzes the *latent variables* of that specific text, then materializes bespoke controls (sliders, dials, toggles) tuned to that exact content. The interface is not pre-built; it is hallucinated on-demand by the model for single-use purposes.

This is not a chatbot. This is a generative cockpit.

---

## Architecture

### Monorepo Layout

```
liquid/
├── apps/
│   ├── web/      # Next.js 16 frontend (CopilotKit + React 19 + Tailwind 4)
│   └── agent/    # LangGraph agent backend (TypeScript, port 8123)
├── turbo.json
└── pnpm-workspace.yaml
```

### Tech Stack

| Layer | Technology |
|---|---|
| Frontend | Next.js 16 (App Router), React 19, Tailwind CSS 4 |
| AI Integration | CopilotKit 1.51.4 (`@copilotkit/react-core`, `@copilotkit/react-ui`, `@copilotkit/runtime`) |
| Agent Framework | LangGraph 1.0.2 (TypeScript) |
| LLM | Claude (via Anthropic SDK — **replace the OpenAI default**) |
| Cache / Realtime | **Upstash Redis** (KV, Streams, Pub/Sub) + **Upstash Vector** (semantic cache) |
| Validation | Zod |
| Monorepo | Turborepo + pnpm workspaces |

### Data Flow

```
User pastes content
       │
       ▼
[Analyst Prompt] ──► Claude ──► strict JSON: { scalars: [...], toggles: [...] }
       │
       ▼
[Builder] CopilotKit Generative UI ──► <ControlPanel /> spawns widgets dynamically
       │
       ▼
User moves "Blame Assignment" dial ──► value injected into Claude system prompt
       │
       ▼
Claude rewrites text ──► streamed back, glass pane morphs
```

---

## Core Components to Build

### 1. Analyst — `apps/agent/src/analyst.ts`

A dedicated LangGraph node that runs first on every new paste.

**System prompt contract (rigid, enforced by Zod):**

```
Analyze the provided text. Output ONLY valid JSON matching this schema:
{
  "scalars": [
    { "id": string, "label": string, "description": string, "default": number (0-100) }
  ],  // 3-5 items
  "toggles": [
    { "id": string, "label": string, "description": string, "default": boolean }
  ]   // 2-3 items
}

Name variables creatively and specifically to THIS text.
Prefer evocative names ("Blame Assignment", "Passive Aggressive") over generic ones ("Tone", "Length").
```

**Validation:** Parse with Zod before forwarding downstream. Retry once on schema failure.

### 2. Builder — `apps/web/src/components/ControlPanel.tsx`

Receives the JSON schema, renders the widgets.

- Map `scalars` → custom dial/slider components
- Map `toggles` → toggle switch components
- Each control dispatches state updates via `useCoAgent()` shared state
- The panel should animate in (glass-shatters-and-reassembles effect) using Tailwind transitions

### 3. Rewriter — `apps/agent/src/rewriter.ts`

Invoked on every control change (debounced 300ms on the frontend).

**System prompt injection pattern:**

```
Rewrite the following text.
Apply these parameters exactly:
{{#each scalars}}
- {{label}}: {{value}}%
{{/each}}
{{#each activeToggles}}
- {{label}}: ENABLED
{{/each}}

Return ONLY the rewritten text, no commentary.
```

### 4. Redis Layer — `apps/agent/src/redis/`

Four distinct Redis patterns, each solving a real problem in the UX. Uses **Upstash Redis** + **Upstash Vector** — no Docker, deploys to Vercel, free tier on both.

**Clients:**
- `@upstash/redis` — KV, Hashes, Streams, session store (both `apps/agent` and `apps/web`)
- `@upstash/vector` — semantic cache with cosine similarity search (`apps/agent` only)

---

#### Strategy 1: Pre-generation KV Cache — `redis/timemachine.ts`

The "Time Machine." Eliminates slider latency by speculatively computing outputs before the user asks.

When the Analyst finishes and controls first appear, fire background jobs in parallel for:
- Every scalar at `0`, `50`, `100` (3 values × N sliders)
- All toggle permutations (cap at 8 combinations when >3 toggles)

**Key schema:** `lc:v:{contentHash}:{paramHash}` → `rewritten_text` (TTL: 1 hour)

```typescript
// contentHash = SHA256 of input text
// paramHash   = SHA256 of JSON.stringify(sortedControlValues)
const key = `lc:v:${contentHash}:${paramHash}`;
await redis.set(key, rewrittenText, { EX: 3600 });
```

On slider drag: hash the nearest quantized state (round to nearest 10), check cache first, return instantly if hit. Fire live LLM call in parallel — swap when it resolves if output differs.

**Cache hit target: >80% of slider moves after the first 5 seconds.**

---

#### Strategy 2: Semantic Cache — `redis/semantic.ts`

The expensive call is the Analyst (parsing raw text into controls). Cache it semantically so the same *type* of text never gets re-analyzed.

Uses **Redis Vector Search** (`FT.SEARCH` with `KNN`).

**Flow:**
1. On paste, embed the input text via `claude-3-haiku` or `text-embedding-3-small` (fast, cheap)
2. Query Redis: find the nearest cached analysis with cosine similarity > 0.92
3. If hit: return cached `ControlSchema` instantly — no LLM call needed
4. If miss: run Analyst, store embedding + schema in Redis


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ostsam) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->

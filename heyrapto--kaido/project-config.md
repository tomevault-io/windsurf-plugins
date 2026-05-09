---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Architecture

**Kaido** is a Next.js (App Router) + Elysia domain/project name generator. Given a project idea, a competitor's name, or an existing name to riff on, it uses the Gemini API to produce tasteful, non-boring name candidates, then checks each for `.com` availability via the API-Ninjas domain endpoint. It retries with entirely new name sets until it surfaces 3 available, good-looking names. Speed is the primary UX goal — all generation and availability checks are parallelised.

### Directory layout

```
app/
├── api/
│   ├── generate/route.ts        # POST — calls Gemini, returns name candidates
│   └── availability/route.ts    # POST — checks domain availability via API-Ninjas
├── layout.tsx                   # Root layout with ZustandProvider + font variables
├── page.tsx                     # Single-page UI (input + results)
├── globals.css                  # CSS variables, font-face imports, base reset
├── components/
│   ├── SearchInput.tsx          # Tabbed input card (idea / competitor / seed)
│   ├── ResultCard.tsx           # Single domain result (name + availability pill)
│   ├── ResultsGrid.tsx          # Animated grid of ResultCards
│   ├── ExampleChips.tsx         # Quick-fill suggestion chips below the search card
│   └── ui/                      # Button, Spinner, EmptyState, ErrorBanner
├── store/
│   └── kaido.ts                 # Zustand store: input, results, loading, error state
├── hooks/
│   └── useNameSearch.ts         # Orchestration hook: generate → check → retry loop
├── lib/
│   ├── gemini.ts                # Gemini API client (gemini-2.5-flash, GoogleGenAI SDK)
│   ├── domain.ts                # API-Ninjas domain availability wrapper
│   └── quality.ts               # Name quality filter (rejects TechBoostifyly-style names)
└── config/
    └── prompts.ts               # Gemini system + user prompt templates
```

### Core flow

1. User submits an **idea string**, a **competitor name**, or a **seed name** (toggled via tabs).
2. `useNameSearch` calls `POST /api/generate` → Gemini returns 8 raw name candidates as a JSON array.
3. `quality.ts` filters out names that fail the non-boring heuristic (see below).
4. Filtered names are immediately rendered as cards in a **checking** state.
5. All domain checks fire in parallel via `Promise.all` — cards update to `available` or `taken` as results arrive.
6. If fewer than 3 names come back available, the hook calls `/api/generate` again with all previous names passed as negative context so Gemini never repeats itself.
7. Loop continues until 3 available names are found or a max-retry limit (5) is hit.
8. A retry note below the grid tells the user how many rounds it took.

### State management

A single Zustand store (`app/store/kaido.ts`) owns all state:

| Field | Type | Purpose |
|---|---|---|
| `query` | `string` | Current user input |
| `queryType` | `'idea' \| 'competitor' \| 'seed'` | Selected input mode |
| `results` | `DomainResult[]` | All name cards surfaced so far |
| `status` | `'idle' \| 'generating' \| 'checking' \| 'done' \| 'error'` | UI phase |
| `attempts` | `number` | Retry count (shown in the retry note) |
| `allTried` | `string[]` | Accumulates every name ever generated to exclude on retry |
| `error` | `string \| null` | Error message |

Never call Gemini or API-Ninjas directly from components — go through the Zustand actions or `useNameSearch`.

### AI — Gemini

- **Model**: `gemini-2.5-flash` (fast, free tier sufficient)
- **SDK**: `@google/genai` — import `GoogleGenAI` from it
- **Key**: set as `GEMINI_API_KEY` in `.env.local`; never hardcode
- All prompt logic lives in `app/config/prompts.ts`; the route handler in `app/api/generate/route.ts` is a thin wrapper
- Ask Gemini to return **JSON only** — a lowercase array of plain strings, no explanations, no markdown fences
- Pass previously generated names in the prompt as `exclude` so retries are genuinely different

Prompt shape (from `prompts.ts`):

```ts
const ctx = {
  idea:       `The user has a project idea: "${input}"`,
  competitor: `The user wants names with a similar feel to the brand: "${input}"`,
  seed:       `The user likes the vibe of the word/name "${input}" and wants related names`,
};

export function buildPrompt(input: string, type: QueryType, exclude: string[]): string {
  return `
You are a naming expert with exceptional taste. Generate 8 short, memorable project or startup names.

${ctx[type]}

Strict rules:
- 1–2 syllables preferred (3 max)
- No portmanteaus with "ify", "ly", "pro", "boost", "tech", "quantum", "ai" unless genuinely elegant
- Think: Notion, Linear, Vercel, Arc, Zed, Loom, Figma, Gleam, Kaido — that register
- Real words, invented words, or portmanteaus that feel human and pronounceable
- No numbers, no hyphens, 3–10 characters each
${exclude.length ? `Do NOT suggest any of these (already tried): ${exclude.join(", ")}` : ""}

Return ONLY a JSON array of lowercase strings. No explanation. No markdown fences.
Example: ["luna","drift","fern"]
  `.trim();
}
```

### Domain availability — API-Ninjas


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [heyrapto/kaido](https://github.com/heyrapto/kaido) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->

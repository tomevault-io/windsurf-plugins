---
trigger: always_on
description: Agentic video editor that watches a folder for new `.mp4` recordings, then runs a 15-stage editing pipeline: ingestion → transcription → silence removal → captions → caption burning → shorts → medium clips → chapters → summary → social media → short posts → medium clip posts → queue build → blog → git push.
---

# Copilot Instructions — vidpipe

## Project Overview

Agentic video editor that watches a folder for new `.mp4` recordings, then runs a 15-stage editing pipeline: ingestion → transcription → silence removal → captions → caption burning → shorts → medium clips → chapters → summary → social media → short posts → medium clip posts → queue build → blog → git push.

**Tech stack:** Node.js, TypeScript (ES2022), ESM modules (`"type": "module"`), `@github/copilot-sdk` for AI agents, OpenAI Whisper for transcription, FFmpeg for all video/audio operations, Winston for logging, Chokidar for file watching, Exa for web search, Sharp for image analysis, Commander for CLI.

## Architecture

### Pipeline Stages (pipeline.ts)

15 stages executed in order. Each is wrapped in `runStage()` which catches errors and records timing. A stage failure does **NOT** abort the pipeline — subsequent stages proceed with whatever data is available.

| # | Stage enum | What it does |
|---|-----------|-------------|
| 1 | `ingestion` | Copy video to `recordings/{slug}/`, extract metadata (duration, size) via ffprobe |
| 2 | `transcription` | Extract audio as MP3 (64kbps mono), send to OpenAI Whisper, chunk if >25MB, merge results |
| 3 | `silence-removal` | Detect silence via FFmpeg, agent decides which regions to cut, `singlePassEdit()` trims video |
| 4 | `captions` | Generate SRT/VTT/ASS from adjusted transcript (no AI needed) |
| 5 | `caption-burn` | Burn ASS subtitles into video; uses `singlePassEditAndCaption()` (one re-encode pass) when silence was removed, or `burnCaptions()` standalone |
| 6 | `shorts` | Agent plans 15–60s short clips, extracts them, generates platform variants (portrait/square/feed), burns captions + portrait hook overlay |
| 7 | `medium-clips` | Agent plans 60–180s medium clips, extracts with xfade transitions for composites, burns captions with medium style |
| 8 | `chapters` | Agent identifies topic boundaries, writes chapters in 4 formats: JSON, YouTube timestamps, Markdown, FFmpeg metadata |
| 9 | `summary` | Agent captures key frames + writes narrative README.md with brand voice, shorts table, chapters section |
| 10 | `social-media` | Agent generates posts for 5 platforms (TikTok, YouTube, Instagram, LinkedIn, X) with web search for links |
| 11 | `short-posts` | For each short clip, agent generates per-platform social posts saved to `shorts/{slug}/posts/` |
| 12 | `medium-clip-posts` | For each medium clip, reuses short-post agent, saves to `medium-clips/{slug}/posts/` |
| 13 | `queue-build` | Copies social posts + video variants into flat `publish-queue/` folder for review |
| 14 | `blog` | Agent writes dev.to-style blog post (800–1500 words) with frontmatter, web search for links |
| 15 | `git-push` | `git add -A && git commit && git push` for the recording folder |

**Key data flow:**
- **Adjusted transcript** (post silence-removal) is used for captions (aligned to edited video)
- **Original transcript** is used for shorts, medium clips, and chapters (they reference original video timestamps)
- Shorts and chapters are generated before summary so the README can reference them

### LLM Provider Abstraction (src/providers/)

All LLM interactions go through a provider abstraction layer. `BaseAgent` accepts an `LLMProvider` instead of directly using `CopilotClient`, allowing agents to work with any supported backend.

| File | Purpose |
|------|---------|
| `types.ts` | `LLMProvider`, `LLMSession`, `LLMResponse`, `TokenUsage`, `CostInfo` interfaces |
| `CopilotProvider.ts` | GitHub Copilot SDK backend (default) |
| `OpenAIProvider.ts` | Direct OpenAI API backend |
| `ClaudeProvider.ts` | Direct Anthropic API backend |
| `index.ts` | Factory — `getProvider()` reads `LLM_PROVIDER` env var, caches singleton, falls back to copilot |

**Cost tracking** (`src/L3-services/costTracking/costTracker.ts`): Singleton `CostTracker` records every LLM call's token usage and cost. At pipeline end, `formatReport()` logs totals and breakdowns by provider, agent, and model. Cost is in USD for OpenAI/Claude and premium requests (PRUs) for Copilot.

### Agent Pattern (@github/copilot-sdk)

All AI agents extend `BaseAgent` (src/agents/BaseAgent.ts):

```typescript
class MyAgent extends BaseAgent {
  constructor() {
    super('MyAgent', SYSTEM_PROMPT)
  }

  protected getTools(): Tool<unknown>[] {
    return [{
      name: 'my_tool',
      description: '...',
      parameters: { /* JSON Schema */ },
      handler: async (args) => this.handleToolCall('my_tool', args as Record<string, unknown>),
    }]
  }

  protected async handleToolCall(toolName: string, args: Record<string, unknown>): Promise<unknown> {
    // Process tool call, store results on the instance
    return { success: true }
  }
}
```

**Flow:** `CopilotClient({ autoStart: true })` → `createSession({ systemMessage, tools, streaming: true })` → `sendAndWait(prompt, 300_000)` → LLM calls tools → agent stores structured results → caller reads them after `run()` completes → `destroy()` tears down session + client.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [htekdev/vidpipe](https://github.com/htekdev/vidpipe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->

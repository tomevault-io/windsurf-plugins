---
trigger: always_on
description: Use Google Gemini with TanStack AI — text, image generation via Imagen and Gemini native (NanoBanana), and experimental TTS via @tanstack/ai-gemini.
---


The Google Gemini adapter provides access to Google's Gemini models, including text generation, image generation with both Imagen and Gemini native image models (NanoBanana), and experimental text-to-speech.

For a full working example with image generation, see the media generation example app.

## Installation

```bash
npm install @tanstack/ai-gemini
```

## Basic Usage

```typescript
import { chat } from "@tanstack/ai";
import { geminiText } from "@tanstack/ai-gemini";

const stream = chat({
  adapter: geminiText("gemini-2.5-pro"),
  messages: [{ role: "user", content: "Hello!" }],
});
```

## Basic Usage - Custom API Key

```typescript
import { chat } from "@tanstack/ai";
import { createGeminiChat } from "@tanstack/ai-gemini";

const adapter = createGeminiChat(process.env.GEMINI_API_KEY!, {
  // ... your config options
});

const stream = chat({
  adapter: adapter("gemini-2.5-pro"),
  messages: [{ role: "user", content: "Hello!" }],
});
```

## Configuration

```typescript
import { createGeminiChat, type GeminiChatConfig } from "@tanstack/ai-gemini";

const config: Omit<GeminiChatConfig, 'apiKey'> = {
  baseURL: "https://generativelanguage.googleapis.com/v1beta", // Optional
};

const adapter = createGeminiChat(process.env.GEMINI_API_KEY!, config);
```
  

## Example: Chat Completion

```typescript
import { chat, toServerSentEventsResponse } from "@tanstack/ai";
import { geminiText } from "@tanstack/ai-gemini";

export async function POST(request: Request) {
  const { messages } = await request.json();

  const stream = chat({
    adapter: geminiText("gemini-2.5-pro"),
    messages,
  });

  return toServerSentEventsResponse(stream);
}
```

## Example: With Tools

```typescript
import { chat, toolDefinition } from "@tanstack/ai";
import { geminiText } from "@tanstack/ai-gemini";
import { z } from "zod";

const getCalendarEventsDef = toolDefinition({
  name: "get_calendar_events",
  description: "Get calendar events for a date",
  inputSchema: z.object({
    date: z.string(),
  }),
});

const getCalendarEvents = getCalendarEventsDef.server(async ({ date }) => {
  // Fetch calendar events
  return { events: [] };
});

const stream = chat({
  adapter: geminiText("gemini-2.5-pro"),
  messages,
  tools: [getCalendarEvents],
});
```

## Model Options

Gemini supports various model-specific options:

```typescript
const stream = chat({
  adapter: geminiText("gemini-2.5-pro"),
  messages,
  modelOptions: {
    maxOutputTokens: 2048,
    temperature: 0.7,
    topP: 0.9,
    topK: 40,
    stopSequences: ["END"],
  },
});
```

### Thinking

Enable thinking for models that support it:

```typescript
modelOptions: {
  thinking: {
    includeThoughts: true,
  },
}
```

### Structured Output

Configure structured output format:

```typescript
modelOptions: {
  responseMimeType: "application/json",
}
```

## Summarization

Summarize long text content:

```typescript
import { summarize } from "@tanstack/ai";
import { geminiSummarize } from "@tanstack/ai-gemini";

const result = await summarize({
  adapter: geminiSummarize("gemini-2.5-pro"),
  text: "Your long text to summarize...",
  maxLength: 100,
  style: "concise", // "concise" | "bullet-points" | "paragraph"
});

console.log(result.summary);
```

## Image Generation

The Gemini adapter supports two types of image generation:

- **Gemini native image models** (NanoBanana) — Use the `generateContent` API with models like `gemini-3.1-flash-image-preview`. These support extended resolution tiers (1K, 2K, 4K) and aspect ratio control.
- **Imagen models** — Use the `generateImages` API with models like `imagen-4.0-generate-001`. These are dedicated image generation models with WIDTHxHEIGHT sizing.

The adapter automatically routes to the correct API based on the model name — models starting with `gemini-` use `generateContent`, while `imagen-` models use `generateImages`.

### Example: Gemini Native Image Generation (NanoBanana)

From the media generation example app:

```typescript
import { generateImage } from "@tanstack/ai";
import { geminiImage } from "@tanstack/ai-gemini";

const result = await generateImage({
  adapter: geminiImage("gemini-3.1-flash-image-preview"),
  prompt: "A futuristic cityscape at sunset",
  numberOfImages: 1,
  size: "16:9_4K",
});

console.log(result.images);
```

### Example: Imagen

```typescript
import { generateImage } from "@tanstack/ai";
import { geminiImage } from "@tanstack/ai-gemini";

const result = await generateImage({
  adapter: geminiImage("imagen-4.0-generate-001"),
  prompt: "A futuristic cityscape at sunset",
  numberOfImages: 1,
});

console.log(result.images);
```

### Image Size Options

#### Gemini Native Models (NanoBanana)

Gemini native image models use a template literal size format combining aspect ratio and resolution tier:

```typescript
// Format: "aspectRatio_resolution"
size: "16:9_4K"
size: "1:1_2K"
size: "9:16_1K"
```

| Component | Values |
|-----------|--------|
| Aspect Ratio | `1:1`, `2:3`, `3:2`, `3:4`, `4:3`, `9:16`, `16:9`, `21:9` |
| Resolution | `1K`, `2K`, `4K` |

#### Imagen Models

Imagen models use WIDTHxHEIGHT format, which maps to aspect ratios internally:

| Size | Aspect Ratio |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [skilld-dev/vue-ecosystem-skills](https://github.com/skilld-dev/vue-ecosystem-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->

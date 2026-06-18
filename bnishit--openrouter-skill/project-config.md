---
trigger: always_on
description: Connect apps to 300+ AI models through OpenRouter's OpenAI-compatible API — model discovery, image generation, multimodal chat (text, images, PDFs), exact per-generation cost tracking, provider routing with fallbacks, tool calling with safety limits, structured output validation, free-model filtering, starter templates (Next.js, Express), asset workflows (icons, OG images), production playbooks, and verification scripts. Use when an agent needs to add or debug OpenRouter usage, build a model pic
---


# OpenRouter Integration

Use official OpenRouter docs as the source of truth for current endpoints, parameters, and capability metadata. Prefer `openrouter.ai/docs`, `openrouter.ai/openapi.json`, and the API reference pages under `openrouter.ai/docs/api-reference`.

## Quick Snippets

Use these for fast copy-paste before reaching for the fuller references or templates.

### Curl: list models

```bash
curl -s https://openrouter.ai/api/v1/models \
  -H "Authorization: Bearer $OPENROUTER_API_KEY" \
  -H "Accept: application/json"
```

### Curl: list providers

```bash
curl -s https://openrouter.ai/api/v1/providers \
  -H "Authorization: Bearer $OPENROUTER_API_KEY" \
  -H "Accept: application/json"
```

### Curl: fetch one generation and its cost

```bash
curl -s "https://openrouter.ai/api/v1/generation?id=$GENERATION_ID" \
  -H "Authorization: Bearer $OPENROUTER_API_KEY" \
  -H "Accept: application/json"
```

### Fetch: free models from the catalog

```ts
const res = await fetch("https://openrouter.ai/api/v1/models", {
  headers: {
    Authorization: `Bearer ${process.env.OPENROUTER_API_KEY}`,
    Accept: "application/json",
  },
});

const json = await res.json();
const freeModels = (json?.data ?? []).filter((model: any) => {
  const pricing = model?.pricing ?? {};
  return ["prompt", "completion", "request", "image"].every((key) => {
    const value = pricing[key];
    return value == null || value === "0";
  });
});
```

### Curl: text-only chat call

```bash
curl -s https://openrouter.ai/api/v1/chat/completions \
  -H "Authorization: Bearer $OPENROUTER_API_KEY" \
  -H "Content-Type: application/json" \
  -H "HTTP-Referer: ${OPENROUTER_SITE_URL:-http://localhost:3000}" \
  -H "X-OpenRouter-Title: ${OPENROUTER_APP_NAME:-My App}" \
  -d '{
    "model": "openai/gpt-4o-mini",
    "messages": [
      {"role": "user", "content": "Write a one-line summary of invoice OCR."}
    ],
    "temperature": 0
  }'
```

### Fetch: image input

```ts
const res = await fetch("https://openrouter.ai/api/v1/chat/completions", {
  method: "POST",
  headers: {
    Authorization: `Bearer ${process.env.OPENROUTER_API_KEY}`,
    "Content-Type": "application/json",
    "HTTP-Referer": process.env.OPENROUTER_SITE_URL || "http://localhost:3000",
    "X-OpenRouter-Title": process.env.OPENROUTER_APP_NAME || "My App",
  },
  body: JSON.stringify({
    model: "google/gemini-2.5-flash",
    messages: [
      {
        role: "user",
        content: [
          { type: "text", text: "Extract all visible text from this image." },
          {
            type: "image_url",
            image_url: { url: imageDataUrl },
          },
        ],
      },
    ],
    temperature: 0,
  }),
});

const json = await res.json();
const content = json?.choices?.[0]?.message?.content;
```

### Fetch: image generation

```ts
const res = await fetch("https://openrouter.ai/api/v1/chat/completions", {
  method: "POST",
  headers: {
    Authorization: `Bearer ${process.env.OPENROUTER_API_KEY}`,
    "Content-Type": "application/json",
    "HTTP-Referer": process.env.OPENROUTER_SITE_URL || "http://localhost:3000",
    "X-OpenRouter-Title": process.env.OPENROUTER_APP_NAME || "My App",
  },
  body: JSON.stringify({
    model: "google/gemini-3.1-flash-image-preview",
    messages: [
      {
        role: "user",
        content: "Generate a clean product-style illustration of a glass teacup on a plain background.",
      },
    ],
    modalities: ["image", "text"],
    image_config: { size: "1024x1024" },
  }),
});

const json = await res.json();
const imageUrl = json?.choices?.[0]?.message?.images?.[0]?.image_url?.url;
```

### Fetch: PDF input with file-parser

```ts
const res = await fetch("https://openrouter.ai/api/v1/chat/completions", {
  method: "POST",
  headers: {
    Authorization: `Bearer ${process.env.OPENROUTER_API_KEY}`,
    "Content-Type": "application/json",
    "HTTP-Referer": process.env.OPENROUTER_SITE_URL || "http://localhost:3000",
    "X-OpenRouter-Title": process.env.OPENROUTER_APP_NAME || "My App",
  },
  body: JSON.stringify({
    model: "google/gemini-2.5-flash",
    messages: [
      {
        role: "user",
        content: [
          { type: "text", text: "Extract the invoice totals as JSON." },
          {
            type: "file",
            file: {
              filename: "invoice.pdf",
              file_data: pdfDataUrl,
            },
          },
        ],
      },
    ],
    plugins: [
      {
        id: "file-parser",
        pdf: { engine: "pdf-text" },
      },
    ],
    response_format: { type: "json_object" },
    temperature: 0,
  }),
});
```

## Workflow

1. Check the docs before making non-trivial changes.
   - Run `scripts/check_openrouter_docs.py --quick` when accuracy matters or the integration seems stale.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bnishit/openrouter-skill](https://github.com/bnishit/openrouter-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

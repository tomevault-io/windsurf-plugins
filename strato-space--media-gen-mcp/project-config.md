---
trigger: always_on
description: **media-gen-mcp** is a Model Context Protocol (MCP) server providing:
---

# AGENTS.md

## Project Overview

**media-gen-mcp** is a Model Context Protocol (MCP) server providing:

- OpenAI Images tools (`gpt-image-1.5`, `gpt-image-1`) for image generation and edits/inpainting.
- OpenAI Videos (Sora) job tooling (`sora-2`, `sora-2-pro`) for video generation and asset downloads.
- Local/URL fetching for images, videos, and documents with optional compression and video input preprocessing via `sharp`.

The server is designed for production use with strict TypeScript compilation, comprehensive error handling, and flexible output formatting for different MCP clients.

## Architecture

```
src/
├── index.ts              # MCP server entry point + tool registrations
└── lib/
    ├── compression.ts    # Image compression + format detection (sharp-based)
    ├── env.ts            # Env parsing + dir/url allowlists (+ glob support)
    ├── helpers.ts        # URL/path validation, output resolution, result building
    ├── logger.ts         # Structured logger + truncation helpers
    └── schemas.ts        # Zod schemas for all tools (exported for tests)
test/
├── compression.test.ts
├── env.test.ts
├── fetch-images.integration.test.ts
├── helpers.test.ts
├── logger.test.ts
└── schemas.test.ts
```

## Tools

| Tool | Purpose | OpenAI API |
|------|---------|------------|
| `openai-images-generate` | Generate images from text prompts | `images.generate` |
| `openai-images-edit` | Edit/inpaint/outpaint images (1–16 inputs) | `images.edit` |
| `openai-videos-create` | Create a video job | `videos.create` |
| `openai-videos-remix` | Remix an existing video job | `videos.remix` |
| `openai-videos-list` | List video jobs | `videos.list` |
| `openai-videos-retrieve` | Retrieve a video job | `videos.retrieve` |
| `openai-videos-delete` | Delete a video job | `videos.delete` |
| `openai-videos-retrieve-content` | Retrieve job assets (video/thumbnail/spritesheet) | `videos.downloadContent` |
| `fetch-images` | Fetch & compress images from URLs/files | None |
| `fetch-videos` | Fetch/list videos from URLs/files | None |
| `fetch-document` | Fetch documents from URLs/files | None |
| `test-images` | Debug MCP result format | None |

## Key Design Decisions

### 1. Mostly single-file server
Most server logic lives in `src/index.ts` for simplicity and ease of review. `src/lib/*` contains small, focused helpers and shared schemas.

### 2. Tool result format (MCP-first)
Tools return MCP `CallToolResult` with:

- `content[]` blocks (`text`, `image`, `resource_link`, `resource`)
- optional `structuredContent` for machine-readable OpenAI responses
- `isError: true` with a single `text` block for failures

For image tools, two parameters standardize output shapes:

- **`tool_result`** (`resource_link` | `image`, default: `resource_link`): Controls `content[]` shape
  - `resource_link`: Emits `ResourceLink` items with `file://` or `https://` URIs
  - `image`: Emits base64 `ImageContent` blocks

- **`response_format`** (`url` | `path` | `b64_json`, default: `url`): Controls `structuredContent` shape for image tools
  - `structuredContent` always contains OpenAI ImagesResponse format
  - `url`: `data[].url` contains file URLs
  - `path`: `data[].path` contains local filesystem paths
  - `b64_json`: `data[].b64_json` contains base64 data

For video download tools, `tool_result` controls `content[]` shape:

- **`tool_result`** (`resource_link` | `resource`, default: `resource_link`)
  - `resource_link`: Emits `ResourceLink` items with `file://` or `https://` URIs
  - `resource`: Emits `EmbeddedResource` blocks with base64 `resource.blob`

For document downloads (`fetch-document`), `tool_result` mirrors video behavior:

- **`tool_result`** (`resource_link` | `resource`, default: `resource_link`)
  - `resource_link`: Emits `ResourceLink` items with `file://` or `https://` URIs
  - `resource`: Emits `EmbeddedResource` blocks with base64 `resource.blob`

For Google video tools, `response_format` controls `structuredContent.response.generatedVideos[].video` fields (`uri` vs `videoBytes`) and remains `url` | `b64_json`.

Per MCP spec 5.2.6, a `TextContent` block with serialized JSON (URLs in `data[]`) is also included for backward compatibility.

### 3. Directory + URL safety model
- Local reads/writes are restricted to `MEDIA_GEN_DIRS` (supports glob patterns).
- Remote downloads are restricted by `MEDIA_GEN_URLS` when set.
- Public URLs for `resource_link` are derived from `MEDIA_GEN_MCP_URL_PREFIXES` matched positionally to `MEDIA_GEN_DIRS`.

### 4. Output file naming
When a tool writes outputs, the default naming is:

`output_<time_t>_media-gen__<tool>_<id>.<ext>`

- Images and `fetch-images` use a generated UUID for `<id>`.
- Videos use the OpenAI `video_id` for `<id>`.
- Documents use a generated UUID for `<id>` (default naming) or the supplied `file` base path.

`fetch-images` and `fetch-videos` also support an `ids` input to retrieve existing local outputs by ID (matching filenames containing `_{id}_` or `_{id}.` under `MEDIA_GEN_DIRS[0]`). IDs are validated to avoid path/glob injection (no `..`, `*`, `?`, or slashes).

For output location overrides:
- OpenAI tools always write under `MEDIA_GEN_DIRS[0]` (no `file` parameter).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [strato-space/media-gen-mcp](https://github.com/strato-space/media-gen-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->

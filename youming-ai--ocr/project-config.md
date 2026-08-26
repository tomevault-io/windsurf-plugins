---
trigger: always_on
description: OCR is a privacy-first, fully client-side OCR (Optical Character Recognition) application. Users upload an image or PDF, and PaddleOCR PP-OCRv6 small runs completely within the browser via ONNX Runtime Web (WASM). A comparison interface displays the source vs. extracted text. All files remain strictly local; there is no server-side OCR and no external AI/LLM processing. The backend `/api` layer only serves static metadata, health checks, and SEO assets.
---

# Repository Guidelines

## Project Overview
OCR is a privacy-first, fully client-side OCR (Optical Character Recognition) application. Users upload an image or PDF, and PaddleOCR PP-OCRv6 small runs completely within the browser via ONNX Runtime Web (WASM). A comparison interface displays the source vs. extracted text. All files remain strictly local; there is no server-side OCR and no external AI/LLM processing. The backend `/api` layer only serves static metadata, health checks, and SEO assets.

*Note: The project was previously a server-side Jina Reader + DeepSeek SEO tool. All Jina/DeepSeek API endpoints, SSRF validation guards, and server-side rate limits are deprecated or deleted. Ignore stale documentation referencing them.*

## Architecture & Data Flow
The application runs as a Client-Side Rendered (CSR) Single Page Application (SPA) built on React 19, Vite 7, and Hono.

### Client-Side OCR Pipeline
The core OCR flow runs entirely in the browser, orchestrated by `OcrEngine` (`src/lib/ocr/engine.ts`), a lazily-constructed singleton facade:
1. **File Handoff**: The file upload is handled in `src/routes/index.tsx` which calls `setPendingFile(file)` in `src/lib/ocr/scan-input.ts` (a non-reactive, transition-safe in-memory singleton) and navigates to `/scan`.
2. **Scan Route**: `src/routes/scan.tsx` fetches the file using `takePendingFile()` on mount. It utilizes a `startedRef` guard to ensure the pipeline runs exactly once under React StrictMode.
3. **Model Loading & Caching**: `src/lib/ocr/model-loader.ts` downloads `det.onnx`, `rec.onnx`, and `ppocrv6_dict.txt` from `/models/pp-ocrv6-small/` on the first run, caches the buffers in IndexedDB, and initializes the `onnxruntime-web` sessions.
4. **PDF Handling**: If the input is a PDF, `src/lib/ocr/pdf-renderer.ts` renders pages one by one using a PDF.js worker thread to bound peak memory usage.
5. **Preprocessing**: `src/lib/ocr/preprocessor.ts` resizes the image, snaps dimensions to multiples of 32 (detector requirement), converts it to a Float32 tensor in CHW format, and normalizes pixel values.
6. **Inference Pipeline**: `src/lib/ocr/pipeline.ts` runs detection and feeds cropped tensors to the recognition model. Direction classification is intentionally not used in the current PP-OCRv6 small deployment.
7. **Postprocessing**: `src/lib/ocr/postprocessor.ts` applies Non-Maximum Suppression (NMS) on bounding boxes, sorts coordinate regions, and decodes recognition scores into character strings using CTC greedy decoding.
8. **Character Translation**: `OcrEngine` translates class indices using the embedded or external character dictionary (`ppocrv6_dict.txt`). The recognition model's class size must strictly match `dict length + 2` (prepended CTC blank index 0 and a trailing space; currently 18710 classes for 18708 dictionary entries).

### Server & Deployment Layers
The Hono backend API (`src/server/hono.ts`) is mounted at `/api` and serves `/health`, `/llm.txt`, `/robots.txt`, and `/sitemap.xml`. It does not contain OCR business logic. Three entrypoints deploy this Hono router:
- **Cloudflare Worker**: `src/worker.ts` handles API routes and serves frontend static assets using Cloudflare Workers Assets via the `ASSETS` binding, configured in `wrangler.toml` with SPA fallback.
- **Bun Self-Host**: `src/prod-server.ts` runs a standalone production server via `Bun.serve`, serving static build assets from `dist/client` and falling back to `index.html` for frontend SPA routing.
- **Local Dev Server**: `src/dev-server.ts` runs a Hono server on port 3001, while Vite (port 5173) handles dev hot-reloading and proxies `/api` requests to port 3001.

## Key Directories
- `src/routes/`: TanStack Router file-based frontend routes (e.g. `index.tsx`, `scan.tsx`).
- `src/server/`: Backend server configurations and routing (`hono.ts`, `worker.ts`, `dev-server.ts`, `prod-server.ts`).
- `src/lib/ocr/`: Core OCR engine, model loader, preprocessing, postprocessing, and PDF renderer.
- `src/components/ocr/`: Frontend OCR UI components (e.g., canvas overlay, upload forms, result lists).
- `src/components/ui/`: UI primitives styled with shadcn/ui.
- `src/styles/`: Tailwind CSS entrypoint (`app.css`) and global theme configurations.
- `src/__tests__/`: Unit tests for pure OCR utility functions.

## Development Commands
Manage and run tasks via Bun:

| Command | Action |
|---|---|
| `bun run dev` | Starts Vite dev server (UI on `:5173`, API proxy on `:3001`) |
| `bun run dev:ui` | Starts Vite frontend dev server only |
| `bun run dev:api` | Starts backend dev server on `:3001` only |
| `bun run build` | Builds frontend assets (`dist/client`) and generates router routes |
| `bun run start` | Runs the production server via `src/prod-server.ts` |
| `bun run deploy` | Vite build followed by `wrangler deploy` to Cloudflare |
| `bun run typecheck` | Runs `tsc --noEmit` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [youming-ai/ocr](https://github.com/youming-ai/ocr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->

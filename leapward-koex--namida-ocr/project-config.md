---
trigger: always_on
description: Namida OCR is a browser extension only. It is not a web app, desktop app, or SaaS product.
---

# AGENTS.md

## Project Identity

Namida OCR is a browser extension only. It is not a web app, desktop app, or SaaS product.

The extension captures a region of the active tab, optionally upscales it, runs OCR locally with bundled OCR assets, copies the recognized Japanese text, and can add furigana or use browser text-to-speech.

Core constraint: keep this project offline-first and serverless. Do not introduce backend services, hosted APIs, telemetry pipelines, auth flows, or any requirement for a project-owned server. Production behavior should come from bundled extension assets and browser-provided capabilities only.

## Browser Support

- Chrome: build with `npm run build:chrome`. Chromium uses the Manifest V3 service worker plus the `offscreen` document flow defined in `manifests/manifest.chrome.json`.
- Edge: Edge support comes from the Chromium build. There is no separate Edge manifest today, so Edge should be treated as a Chromium target that uses the Chrome build output. The popup already has Edge-specific shortcut handling in `src/ui/index.ts`.
- Firefox: build with `npm run build:firefox`. Firefox uses `manifests/manifest.firefox.json`, background scripts, and does not use the Chromium `offscreen` permission/document flow.

When changing permissions, background execution, popup behavior, or shortcut flows, keep Chrome, Edge, and Firefox aligned. If you introduce a Chromium-only API, provide a Firefox-safe path.

## Key Runtime Files

- `src/background/index.ts`: background orchestration, keyboard shortcut handling, OCR/furigana routing, and Chromium offscreen bootstrap.
- `src/background/ocr/OcrService.ts`: OCR backend selection and lifecycle management for offscreen/background OCR.
- `src/background/ocr/TesseractOcrBackend.ts`: bundled Tesseract worker creation and OCR cleanup/scoring behavior.
- `src/background/ocr/ScribeOcrBackend.ts`: experimental `scribe.js-ocr` backend wired for local extension assets only.
- `src/background/ocr/PaddleOnnxOcrBackend.ts`: experimental PaddleOCR ONNX backend using bundled local ONNX assets plus `onnxruntime-web`, preferring bundled WebGPU-capable JSEP runtime assets when available.
- `src/offscreen/index.ts`: Chromium offscreen document entrypoint for OCR and furigana work when the background context cannot host workers directly.
- `src/content/index.ts`: content-side snipping, OCR flow, clipboard, overlay, and floating window behavior.
- `src/ui/index.ts`: popup settings UI, browser-specific shortcut UX, and speech voice availability messaging.
- `webpack.config.js`: browser-specific manifest merge plus local bundling of OCR language data and runtime assets, including browser-specific PaddleOCR bundle selection.

## Offline and No-Server Rules

- OCR, upscaling, and furigana generation are expected to run locally from the extension bundle.
- Keep traineddata, WASM, and model assets bundled locally. Do not switch this project to CDN downloads or server-backed OCR.
- Any `scribe.js-ocr` integration must continue to use extension-local language/model assets. Do not rely on its CDN fallback.
- Any `paddleonnx` integration must continue to use extension-local ONNX, dictionary, manifest, and ONNX Runtime JSEP/WASM assets. Do not rely on remote model fetches or runtime downloads.
- Keep both committed PaddleOCR bundles local to the repo when Chromium/server and Firefox/mobile packaging are supported, but only copy the browser-appropriate bundle into `dist/` at build time.
- Keep the built PaddleOCR metadata file named `libs/paddleocr/paddleocr-manifest.json`; Chrome Web Store uploads must not include nested `manifest.json` files beyond the root extension manifest.
- Do not add any extension feature that requires calling an application server to function.
- Browser/system capabilities such as clipboard access or speech synthesis are acceptable. They are not a substitute for adding project servers.
- The only HTTP server in this repo is `tests/serve-fixtures.mjs`, which exists solely to serve local Playwright fixtures during tests. It is not part of product architecture.

## Build and Test Commands

- `npm run build:chrome`
- `npm run build:firefox`
- `npm run prepare:paddleocr-onnx`: downloads the official PP-OCRv5 server repos, converts them to ONNX, normalizes unsupported `MaxPool ceil_mode` attributes for ONNX Runtime Web acceleration, and refreshes the committed server PaddleOCR bundle metadata for the experimental `paddleonnx` backend.
- `npm run prepare:paddleocr-onnx:mobile`: downloads the official PP-OCRv5 mobile repos, converts them to ONNX, normalizes unsupported `MaxPool ceil_mode` attributes for ONNX Runtime Web acceleration, and refreshes the committed Firefox/mobile PaddleOCR bundle metadata for the experimental `paddleonnx` backend.
- `npm run test:e2e`: builds the Chromium extension with the default OCR model and runs the Playwright suite.
- `npm run test:e2e:tesseract`: runs the Chromium Playwright OCR suite with the bundled `tesseract` backend.
- `npm run test:e2e:scribejs`: runs the Chromium Playwright OCR suite with the experimental `scribejs` backend.
- `npm run test:e2e:paddleonnx`: runs the Chromium Playwright OCR suite with the experimental `paddleonnx` backend.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Leapward-Koex/Namida-OCR](https://github.com/Leapward-Koex/Namida-OCR) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->

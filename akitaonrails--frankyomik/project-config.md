---
trigger: always_on
description: This repo has two moving parts:
---

# Frank Yomik

This repo has two moving parts:

- `server/`: Go API plus Python workers for manga and webtoon processing
- `client/`: Flutter reader app that wraps Kindle and Naver Webtoon in a WebView

If you need to get oriented fast, start with `server/main.go`, `server/handlers.go`, `server/worker/consumer.py`, and `client/lib/screens/reader_screen.dart`.

## What the system does

The API accepts page images, hashes them, deduplicates them, and queues jobs in Redis. Python workers pull from Redis, run OCR and translation, render a new image, cache the result on disk, and publish completion events. The Flutter client watches those events and overlays translated images back into the reader.

There are three pipelines:

- `manga_translate`: Japanese manga page -> English render
- `manga_furigana`: Japanese manga page -> furigana annotations
- `webtoon`: Korean webtoon page -> English render

## Repo map

### Server

- `server/main.go`: boot, env parsing, Redis wiring, graceful shutdown
- `server/handlers.go`: REST API, cache endpoints, metadata patching, health
- `server/middleware.go`: bearer-token auth
- `server/queue.go`: Redis Streams submission and dedup
- `server/results.go`: Redis-backed job status and image retrieval
- `server/cache.go`: disk cache v2, content-addressed objects, manifest/ref layout
- `server/websocket.go`: websocket upgrade and origin checks
- `server/worker/consumer.py`: Redis stream consumer, result publishing, cache writes
- `server/worker/job.py`: pipeline routing and metadata payload generation
- `server/worker/page_cache.py`: Python-side cache v2 writer/reader
- `server/kindle/`: manga OCR, translation, rendering, furigana, bubble detection
- `server/webtoon/`: webtoon OCR, translation, rendering, scraper

### Client

- `client/lib/screens/home_screen.dart`: launcher with arbitrary URL entry
- `client/lib/screens/reader_screen.dart`: main runtime, capture flow, overlay logic
- `client/lib/services/api_service.dart`: HTTP client for job submit/status/image download
- `client/lib/services/websocket_service.dart`: realtime progress/completion feed
- `client/lib/providers/jobs_provider.dart`: job state, cache lookup, polling fallback
- `client/lib/webview/js_bridge.dart`: JS handler registration and strategy selection
- `client/lib/webview/strategies/kindle_strategy.dart`: Kindle page detection and capture
- `client/lib/webview/strategies/naver_webtoon_strategy.dart`: Naver detection and capture

## Runtime flow

### Kindle / manga flow

1. Flutter loads `read.amazon.co.jp` in a WebView.
2. `KindleStrategy` injects JS that watches the visible blob-backed page image.
3. On page detection, `ReaderScreen` captures the visible page.
4. The client submits the PNG to `POST /api/v1/jobs`.
5. Go stores the source image in Redis and the disk object store, then enqueues the job.
6. The worker processes the page with the manga pipeline and writes the rendered image plus metadata to cache v2.
7. The worker stores transient job status in Redis and publishes a websocket notification.
8. The client downloads the result and overlays it on the reader.

### Webtoon flow

1. Flutter loads `comic.naver.com` in a WebView.
2. `NaverWebtoonStrategy` discovers page images and reports them back to Dart.
3. The client captures each image through JS `fetch()` first, then falls back to an app-side HTTP fetch if needed.
4. The worker runs the webtoon pipeline and returns the translated image.

## Cache model

The server and worker share the same cache layout.

- Objects live at `cache/v2/objects/<aa>/<bb>/<sha256>`.
- Page manifests live at `cache/v2/pages/by-hash/<pipeline>/<source_hash>/manifest.json`.
- Optional metadata refs live at `cache/v2/pages/by-ref/<pipeline>/<title_slug>/<chapter>/<page>.json`.

The manifest is the source of truth. Redis is short-lived delivery state, not durable storage.

## API surface

Main endpoints:

- `POST /api/v1/jobs`
- `GET /api/v1/jobs/:id`
- `GET /api/v1/jobs/:id/image`
- `DELETE /api/v1/jobs/:id`
- `GET /api/v1/cache/...`
- `PATCH /api/v1/cache/by-hash/:pipeline/:source_hash/meta`
- `GET /api/v1/health`
- `GET /api/v1/ws`

Everything except `/api/v1/health` is bearer-token protected.

## Local dev

### Server

```bash
redis-server
cd server && AUTH_TOKEN=secret go run .
cd server && python -m worker --pipeline both
```

### Client

```bash
cd client
flutter pub get
flutter run -d linux
```

### Useful tests

```bash
cd server && go test ./...
cd server && pytest tests/unit/test_page_cache.py
cd client && flutter test
```

## Configuration that matters

- `AUTH_TOKEN`: required by the API
- `REDIS_URL`: Redis connection for the API
- `CACHE_DIR`: shared disk cache path for the API
- `server/config.yaml`: worker-side config for Ollama, OCR, fonts, cache, and webtoon settings
- `docker-compose.yml`: the normal multi-service deployment path

## Practical trust boundaries

These are the places worth treating as hostile input:

- image uploads to `POST /api/v1/jobs`
- metadata fields like `title`, `chapter`, and `page_number`
- websocket clients and job subscription lists
- web pages loaded inside the app WebView
- fallback image URLs surfaced by page JS in the client
- anything persisted under `cache/`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [akitaonrails/FrankYomik](https://github.com/akitaonrails/FrankYomik) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->

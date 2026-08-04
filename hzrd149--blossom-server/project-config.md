---
trigger: always_on
description: Guidance for AI coding agents working in this repository.
---

# AGENTS.md — Blossom Server

Guidance for AI coding agents working in this repository.

---

## Project Overview

Blossom Server is a **Deno 2**-based HTTP server implementing the
[Blossom](https://github.com/hzrd149/blossom) blob-storage protocol (BUDs
01/02/04/05/06/09/11). It uses **Hono** for routing, **LibSQL** (embedded
SQLite) for metadata, and supports local-disk and S3 storage. Authentication is
via BUD-11 Nostr signed events (kind 24242). The admin dashboard is server-side
rendered Hono JSX, runs on the main thread, and is protected by HTTP Basic Auth.

---

## Commands

All commands use the **Deno** toolchain. There is no `package.json`.

```sh
# Development (file-watching)
deno task dev

# Production start
deno task start

# Run the full test suite
deno task test

# Run a single test file
deno test --env-file=.env --allow-net --allow-read --allow-write --allow-env --allow-ffi --allow-sys tests/e2e/upload.test.ts

# Run a single test case by name
deno test --filter "PUT /upload returns 200" --env-file=.env --allow-net --allow-read --allow-write --allow-env --allow-ffi --allow-sys tests/

# Lint (Deno built-in)
deno lint

# Format check
deno fmt --check

# Format (auto-fix)
deno fmt

# Pre-build the landing page client bundle (output: public/client.js)
# Required before running the server when the landing page is enabled.
deno task build
```

> **Before every commit:** run `deno fmt` to auto-format all changed files.
> Unformatted code will fail CI. Run `deno fmt --check` to verify without
> modifying files.

> **Read before writing tests:** `TESTING.md` contains the full planned test
> matrix and helper patterns. Tests go in `tests/unit/` (pure logic) or
> `tests/e2e/` (full Hono app via `app.fetch()` — no real HTTP port needed).

---

## Project Structure

```
blossom-server/
├── main.ts                   # Entry: load config → init DB → init pool → buildApp → Deno.serve
├── deno.json                 # Tasks, import map, compiler options
├── config.example.yml        # Annotated reference config (copy to config.yml to run)
├── ARCHITECTURE.md           # Architecture docs — read before structural changes
├── TESTING.md                # Planned test suite — read before writing tests
├── public/
│   ├── favicon.ico
│   └── client.js             # Landing page bundle — built at startup or via deno task build
└── src/
    ├── server.ts             # Hono app assembly: middleware + route routers
    ├── config/
    │   ├── schema.ts         # Zod config schema (all defaults live here)
    │   └── loader.ts         # YAML load + ${ENV_VAR} interpolation + Zod safeParse
    ├── middleware/
    │   ├── auth.ts           # BUD-11 parse, requireAuth(), optionalAuth(), requireXTag()
    │   ├── cors.ts           # BUD-01 CORS
    │   ├── errors.ts         # errorResponse() + global onError handler
    │   ├── debug.ts          # debug() logging helper
    │   └── logger.ts         # requestLogger middleware
    ├── routes/
    │   ├── blobs.ts          # GET/HEAD /:sha256[.ext] (BUD-01)
    │   ├── upload.ts         # PUT /upload, HEAD /upload (BUD-02/06)
    │   ├── delete.ts         # DELETE /:sha256 (BUD-02)
    │   ├── list.ts           # GET /list/:pubkey (BUD-02)
    │   ├── mirror.ts         # PUT /mirror (BUD-04)
    │   ├── media.ts          # PUT /media, HEAD /media (BUD-05)
    │   ├── report.ts         # PUT /report (BUD-09)
    │   ├── admin-router.tsx  # /admin/* SSR pages + action endpoints (main thread)
    │   └── landing.tsx       # GET /, GET /client.js
    ├── admin/                # Admin dashboard SSR components (hono/jsx)
    ├── landing/
    │   ├── client/           # Client-side island (hono/jsx/dom, bundled to public/client.js)
    │   │   ├── index.tsx     # Entry point — hydrates #upload-root
    │   │   ├── App.tsx
    │   │   ├── UploadForm.tsx
    │   │   ├── MirrorForm.tsx
    │   │   └── ...
    │   ├── layout.tsx        # HTML shell + Tailwind CDN
    │   ├── page.tsx          # LandingPage async SSR component
    │   ├── upload-island.tsx # Island mount point (data-* attrs → client hydration)
    │   ├── server-info.tsx
    │   └── stats-bar.tsx
    ├── storage/
    │   ├── interface.ts      # IBlobStorage + WriteSession interfaces
    │   ├── local.ts          # LocalStorage implementation (Deno FS)
    │   └── s3.ts             # S3Storage implementation
    ├── db/
    │   ├── client.ts         # initDb() / getDb() singleton
    │   ├── blobs.ts          # All SQL query functions
    │   ├── handle.ts         # IDbHandle interface
    │   ├── direct.ts         # DirectDbHandle — wraps @libsql/client Client
    │   ├── bridge.ts         # MessageChannel bridge (upload workers → main thread DB)
    │   └── proxy.ts          # Worker-side DbProxy
    ├── optimize/
    │   ├── index.ts          # optimizeMedia() dispatcher
    │   ├── image.ts          # sharp-based image optimization
    │   └── video.ts          # fluent-ffmpeg video transcoding
    ├── prune/
    │   ├── prune.ts          # Prune loop (removeWhenNoOwners / storage rules)
    │   └── rules.ts          # getFileRule() — per-blob rule evaluation
    ├── utils/
    │   ├── mime.ts           # mimeToExt()
    │   ├── streams.ts        # Stream helpers

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hzrd149/blossom-server](https://github.com/hzrd149/blossom-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->

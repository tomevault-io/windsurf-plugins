---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Idswyft is an open-source identity verification platform. Developers integrate via API to verify government-issued IDs (passport, driver's license, national ID) with OCR, cross-validation, liveness detection, and face matching. Self-hostable via Docker Compose or available as a managed cloud service at idswyft.app.

## Repository Structure

```
backend/     Express + TypeScript API server (v1.7.0)         → port 3001
engine/      ML verification engine (TensorFlow, PaddleOCR)   → port 3002
frontend/    Vite + React + TypeScript (edition-aware)         → port 5173 (dev)
sdks/javascript/  npm SDK (idswyft-sdk v4.0.0)
docker-compose.yml  4-container self-hosted stack
install.sh          Interactive setup script
```

## Technical Stack

| Component | Technology |
|-----------|-----------|
| Backend | Express 4, TypeScript 5, ESM modules |
| Database | PostgreSQL 16 (via Supabase JS client + direct `pg` for migrations) |
| Frontend | React 18, Vite 4, Tailwind CSS 3, Zustand, React Query |
| OCR | PaddleOCR (ONNX, primary), Tesseract.js (fallback), optional LLM vision |
| Face Recognition | vladmandic/face-api.js (TF.js + WASM backend) |
| Deepfake Detection | EfficientNet-B0 via ONNX Runtime |
| Tamper Detection | Sharp (ELA, entropy, FFT spectral analysis) |
| Barcode | @zxing/library (PDF417, QR), MRZ parser |
| Storage | Local filesystem or S3-compatible |
| Auth | HMAC-SHA256 API keys, JWT (httpOnly cookies), OTP |

## Architectural Invariant: Deterministic Decisions

**All comparison and decision logic must be deterministic and fully auditable.** No LLM or probabilistic model may be used for any verification decision — only for OCR text extraction, which is isolated behind a provider interface.

- Gates use checksums, exact string matching, Levenshtein distance, cosine similarity with fixed thresholds
- Same inputs must always produce the same verification result
- LLMs may only read text from images (extraction) — never decide pass/fail/review
- The LLM provider interface is isolated in `engine/src/providers/ocr/LLMFieldExtractor.ts` — it must never be imported or called from gate logic, cross-validation, liveness scoring, or face matching

## Verification Pipeline

5-step state machine with automatic gate transitions:

```
AWAITING_FRONT → AWAITING_BACK → CROSS_VALIDATING → AWAITING_LIVE → FACE_MATCHING → COMPLETE
                                                                                   → HARD_REJECTED
```

| Step | Route | What happens |
|------|-------|-------------|
| 1 | `POST /api/v2/verify/initialize` | Create session |
| 2 | `POST /api/v2/verify/:id/front-document` | OCR, face detection, tamper detection |
| 3 | `POST /api/v2/verify/:id/back-document` | Barcode/MRZ + auto cross-validation |
| 4 | `POST /api/v2/verify/:id/live-capture` | Liveness (head-turn or passive) + auto face match |
| 5 | `GET /api/v2/verify/:id/status` | Poll for final result |

**Final results**: `verified`, `failed`, or `manual_review`

## Engine Worker Architecture

The engine (`engine/`) is a separate container (~1.5GB) that handles ML-heavy operations. The API container stays lightweight (~250MB).

- `ENGINE_URL` env var controls routing: set → call engine via HTTP; unset → local fallback functions
- Three endpoints: `POST /extract/front`, `POST /extract/back`, `POST /extract/live`
- Heavy deps (TensorFlow, ONNX, PaddleOCR, canvas) live only in the engine

## AML / Sanctions Screening

Gate 6 screens extracted names against sanctions lists. Configured via:

- `AML_PROVIDER` — comma-separated: `opensanctions`, `offline`, or `none` (default: `none`)
- `OFAC_SDN_PATH` — path to local OFAC SDN CSV file (used when `offline` provider is active)
- `OFAC_AUTO_LOAD=true` — download OFAC SDN from US Treasury at startup (alternative to local file)
- `developers.aml_enabled` — per-developer toggle (default: `true`), managed via Settings API

## Auth System

6 auth mechanisms in `backend/src/middleware/auth.ts`:

| Method | Header/Source | Use case |
|--------|-------------|----------|
| `authenticateAPIKey` | `X-API-Key` header | Developer API calls |
| `authenticateServiceToken` | `X-Service-Token` header | Service-to-service |
| `authenticateJWT` | `idswyft_token` cookie or Bearer | Admin dashboard |
| `authenticateDeveloperJWT` | `idswyft_token` cookie or Bearer | Developer portal |
| `authenticateReviewerJWT` | `idswyft_token` cookie or Bearer | Invited reviewers |
| `authenticateAdminOrReviewer` | Cookie or Bearer | Shared admin endpoints |

**API key format**: `ik_` prefix + 64 hex chars. Keys are HMAC-SHA256 hashed before storage. Sandbox mode is a boolean `is_sandbox` on the key, not a prefix distinction.

## Edition System

Build-time `VITE_EDITION` flag in `frontend/src/config/edition.ts`:
- `community` (default): self-hosted, Dev Portal at `/`, minimal chrome
- `cloud`: managed idswyft.app, marketing homepage at `/`, full navbar/footer

## Docker Architecture

4 containers (+ optional Caddy for HTTPS):

```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [team-idswyft/idswyft-community](https://github.com/team-idswyft/idswyft-community) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->

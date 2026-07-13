---
trigger: always_on
description: Auto-generated from all feature plans. Last updated: 2026-02-22
---

# ag-workspace Development Guidelines

Auto-generated from all feature plans. Last updated: 2026-02-22

## Active Technologies
- TypeScript 5 / Node.js 20 + Next.js 16, React 19, Tailwind CSS 4, Axios, lucide-react (001-frontend-2-migration)
- No local storage; all state served by backend API (001-frontend-2-migration)
- TypeScript 5 / Node.js 20 + Next.js 16.1.6, React 19, Tailwind CSS 4, Axios, lucide-react, clsx (003-social-sso-login)
- Python 3.11 for backend, TypeScript (React/Next.js) for frontend + FastAPI, python-jose, @supabase/supabase-js, @supabase/ssr (001-migrate-supabase)
- Supabase PostgreSQL (001-migrate-supabase)
- Python 3.11 (backend), TypeScript / Node.js 20 (frontend-2) + FastAPI, SQLModel, Celery (Redis broker), pydantic-ai, pypdf, python-jose (002-api-alignment)
- Python 3.11 (backend), TypeScript / Next.js 16 (frontend) — rebrand: project renamed from "Pareto AI" to "Zabt" (001-rebrand-zabt)
- Python 3.11 (Backend), YAML (Docker Compose) + boto3, docker-compose (010-fix-minio-connection)
- MinIO (S3-compatible) (010-fix-minio-connection)
- Python 3.11 (backend), TypeScript / Node.js 20 (frontend-2) + FastAPI, SQLModel, Celery (Redis broker), boto3, pydantic-settings (011-minio-webhook)
- PostgreSQL (via SQLModel), MinIO (S3-compatible object storage) (011-minio-webhook)
- Python 3.11 + Typer (CLI framework), rich (terminal formatting — bundled with Typer), existing transcription stack (openai-whisper, whisperx, pyannote-audio, pydantic-ai) (012-transcription-cli)
- N/A — CLI reads local files; no database or object storage required (012-transcription-cli)
- Python 3.11 + FastAPI, SQLModel, Celery (Redis broker), google-cloud-speech (V2), google-cloud-storage, provider abstraction pattern (TranscriptionProvider Protocol) (013-whisper-chirp-migration)
- PostgreSQL (via SQLModel, unchanged), MinIO (audio uploads, unchanged), GCS (audio staging for Chirp 3 BatchRecognize) (013-whisper-chirp-migration)
- Python 3.11 (backend) + FastAPI, Celery, SQLModel, uv (package manager), Docker BuildKit (014-docker-build-optimization)
- PostgreSQL (via SQLModel), MinIO (S3-compatible object storage) — unchanged (014-docker-build-optimization)
- TypeScript 5 / Node.js 20 + Next.js 16, React 19 + Tailwind CSS 4, clsx, @supabase/supabase-js, @supabase/ssr, lucide-react (015-logout-button)
- N/A — no data model changes; Supabase manages session cookies (015-logout-button)
- Python 3.11 (backend), TypeScript 5 / Node.js 20 (frontend) + FastAPI, Celery (canvas: chain, link_error), SQLModel, Redis; Next.js 16, React 19, Axios (017-transcription-progress)
- PostgreSQL (via SQLModel), Redis (Celery broker + Pub/Sub), MinIO (S3-compatible) (017-transcription-progress)
- TypeScript 5 / Node.js 20 + Next.js 16, React 19, Tailwind CSS 4, lucide-react, clsx, Axios (018-home-ui-redesign)
- N/A — no data model or storage changes (018-home-ui-redesign)
- Python 3.11 (backend), TypeScript 5 / Node.js 20 (frontend) + FastAPI, SQLModel, Celery (Redis broker), pydantic-ai / OpenAI client; Next.js 16, React 19, Tailwind CSS 4, Axios, lucide-react (001-summary-templates)
- PostgreSQL (via SQLModel) — new `summarytemplate` table; `user` and `meeting` tables extended (001-summary-templates)
- TypeScript 5 / Node.js 20 + Next.js 16, React 19, `pdfmake` (new), `@types/pdfmake` (new dev dep) (001-export-summary-pdf)
- YAML (Docker Compose + Kong declarative config); Python 3.11 env var change only — no code changes required + Kong Gateway 3.6 (Docker image `kong:3.6`), existing Docker Compose stack (001-kong-api-gateway)
- MinIO (S3-compatible) — unchanged; access pattern changes (routed through Kong) (001-kong-api-gateway)
- N/A — all analytics data lives in PostHog Cloud (001-product-analytics)
- Python 3.11 (backend only — no frontend changes) + FastAPI 0.129+, Celery 5.6+, SQLModel 0.0.37, OpenAI SDK 2.21+, Logfire 4.25.0 (already installed) (001-observability)
- PostgreSQL via SQLModel (unchanged) (001-observability)
- Python 3.11 (backend), TypeScript / Node.js 20 (frontend-2), YAML (Docker Compose, Kong, Cloudflare) + Docker Compose, Cloudflare Tunnel (`cloudflared`), Kong 3.6, PostgreSQL 16, Redis 7, MinIO, Qdrant, Celery (019-vps-lift-shift)
- PostgreSQL (via SQLModel), MinIO (S3-compatible), Redis (Celery broker), Qdrant (vector DB, empty for now) (019-vps-lift-shift)
- Python 3.11 (backend) + FastAPI, boto3, Celery (Redis broker), pydantic-settings (020-s3-storage-switch)
- PostgreSQL (via SQLModel), MinIO/S3 (object storage) (020-s3-storage-switch)
- Python 3.11 (backend worker + RunPod handler) + FastAPI, Celery (Redis broker), `runpod` Python SDK (new — client calls), WhisperX, pyannote-audio, boto3 (021-runpod-worker)
- PostgreSQL (via SQLModel), S3/MinIO (audio files — presigned URLs passed to RunPod) (021-runpod-worker)
- Python 3.11 (backend), TypeScript 5 / Node.js 20 (frontend) + FastAPI, SQLModel (backend); Next.js 16, React 19, Tailwind CSS 4, Tiptap (new — `@tiptap/react`, `@tiptap/starter-kit`, `@tiptap/extension-link`, `tiptap-markdown`) (001-edit-summary)
- PostgreSQL (via SQLModel) — two new columns on `meeting` table (001-edit-summary)
- TypeScript 5 / Node.js 20 + Next.js 16, React 19 + pdfmake (already installed), existing `pdf-export.ts` utility (001-transcript-pdf-download)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [afeef/zabt-ai](https://github.com/afeef/zabt-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-13 -->

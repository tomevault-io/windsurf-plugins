---
trigger: always_on
description: This file is the standing brief for Claude Code. **Read it before any task.** It defines the product, the stack, the guardrails, and the phased plan. The companion `Takeoff_AI_PRD_Architecture.md` holds the full product rationale. **When they conflict, ask.**
---

# CLAUDE.md — Takeoff.AI Build Spec

This file is the standing brief for Claude Code. **Read it before any task.** It defines the product, the stack, the guardrails, and the phased plan. The companion `Takeoff_AI_PRD_Architecture.md` holds the full product rationale. **When they conflict, ask.**

---

## 1. What we are building

Takeoff.AI — an AI construction takeoff & estimating web app, equivalent in capability to Togal.ai. Estimators upload architectural drawings (PDF/image/CAD); the app auto-detects, measures, labels, and quantifies rooms, walls, openings, fixtures, and trade conditions, then exports estimate-ready quantities. Targets ~5x faster takeoff and 95–98% measurement accuracy.

Build **original** code and models. Match Togal's capability, never copy its brand, UI assets, copy, or trained models.

---

## 2. Non-negotiable architectural guardrails

1. **Vercel hosts UI + orchestration ONLY.** No GPU, ~60–300s function limit. Never run CV inference or model training inside a Vercel route.
2. **All heavy ML runs on a separate GPU service** (Modal / Replicate / RunPod / AWS GPU), invoked asynchronously.
3. **Pattern:** Browser → Vercel (validate + enqueue) → Queue (Inngest/Trigger.dev) → GPU service (infer) → Postgres/PostGIS + S3/R2 → webhook → Vercel → Browser. Vercel routes must return fast; long work is a job.
4. **Geometry is first-class.** Store detections/measurements as real geometry in PostGIS, not as loose JSON blobs.
5. **Log every user correction** (accept/reject/edit) to a `CorrectionEvent` table from day one — it is the training-data flywheel.
6. **Everything AI produces must be human-editable** in the UI.

---

## 3. Tech stack (use these unless told otherwise)

- **Frontend:** Next.js 14 (App Router) + TypeScript + Tailwind + shadcn/ui. State: Zustand + TanStack Query.
- **Drawing canvas:** PDF.js + OpenSeadragon/PixiJS for large-image tiling; Konva/Fabric.js for editable vector overlays.
- **API:** Next.js route handlers (REST or tRPC) on Vercel.
- **Async/queue:** Inngest or Trigger.dev (+ QStash if needed).
- **Auth:** Clerk (orgs/teams built-in) or Auth.js.
- **DB:** Postgres on Neon or Supabase + PostGIS + pgvector. ORM: Prisma.
- **Object storage:** Cloudflare R2 or AWS S3 (presigned uploads).
- **Cache/presence:** Upstash Redis; Liveblocks for real-time collaboration.
- **ML inference host:** Modal or Replicate (start here for speed); RunPod/AWS later.
- **CV models:** SAM2 (zero-shot first), then fine-tuned Mask R-CNN / U-Net / YOLOv8-seg for spaces; YOLOv8/v11 or DETR for symbols; PaddleOCR for text.
- **LLM/RAG (Takeoff.CHAT):** GPT-4o or Claude + pgvector.
- **Payments:** Stripe. **Observability:** Sentry + Vercel Analytics.

---

## 4. Repository structure (Turborepo monorepo)

```
takeoff-ai/
  apps/web/              # Next.js app (Vercel): UI + API routes
  packages/ui/           # shared shadcn components
  packages/db/           # Prisma schema, migrations, PostGIS helpers
  packages/geometry/     # measurement engine (pixels -> sqft/linft/counts)
  packages/config/       # eslint, tsconfig, env schema (zod)
  services/inference/    # GPU: OCR, segmentation, detection (Python)
  services/comparison/   # drawing-diff worker (Python/OpenCV)
  services/jobs/         # Inngest/Trigger functions
  ml/datasets/           # DVC-tracked data pointers
  ml/training/           # fine-tuning scripts/configs
  ml/eval/               # accuracy harness (mIoU, mAP, measurement error)
  ml/registry/           # model cards & versions
  infra/                 # IaC, deploy manifests
  turbo.json  package.json  README.md  CLAUDE.md
```

---

## 5. Core data model (Prisma + PostGIS)

Implement these entities (see PRD §9.1 for fields): `Organization`, `User`, `Project`, `Sheet` (drawing page: fileUrl, pageNo, scale, dpi), `Takeoff`, `Condition` (trade, type, unit), `Detection` (PostGIS `geometry`, class, confidence), `Measurement` (value, unit, geom), `CorrectionEvent` (action, before, after, userId), `ModelVersion`.

**Rules:** multi-tenant by `orgId` with row-level isolation; store `Detection.geom` and `Measurement.geom` as PostGIS geometry; keep raster/source files in object storage, only URLs in Postgres.

---

## 6. Phased build plan (build in this order, ship each phase)

**Phase 0 — Foundation (no AI yet)** Monorepo scaffold; Next.js app on Vercel; Clerk auth + orgs; Neon Postgres + PostGIS + Prisma migrations; R2 bucket + presigned upload; PDF/drawing viewer (pan/zoom/tile); manual takeoff tools (draw polygon/line/count, scale calibration); conditions table; Excel/CSV export. **Done when:** a user can upload a drawing, set scale, measure manually, and export quantities.

**Phase 1 — AI MVP** Stand up GPU inference service (Modal/Replicate) with SAM2 zero-shot room detection; job queue wiring (enqueue → infer → webhook → render); measurement engine converts detections to quantities; render AI detections on canvas with accept/reject. **Done when:** one-click AI takeoff works end-to-end and is editable.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Siddartha-DevOps/TakeOff](https://github.com/Siddartha-DevOps/TakeOff) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-12 -->

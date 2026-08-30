---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Commands

| Task | Command |
|---|---|
| Dev | `bun run dev` |
| Dev compatibility alias | `bun run dev:all` (currently the same Next.js process) |
| Build for production | `bun run build` |
| Start production server | `bun run start` |
| Author preflight | `bun run doctor` |
| Lint | `bun run lint` |
| Test | `bun run test` |
| Test OpenSCAD WASM runtime | `bun run test:wasm` |
| Audit dependency licenses | `bun run license:audit` |
| Check OpenSCAD libraries | `bun run scad:libs:check` |
| Install default OpenSCAD libraries | `bun run scad:libs:install` |
| Install GPL OpenSCAD libraries explicitly | `bun run scad:libs:install:gpl` |
| Sync DB schema to SQLite | `bun run db:push` |
| Generate Prisma client | `bun run db:generate` |
| Run DB migrations | `bun run db:migrate` |
| Reset DB | `bun run db:reset` |
| Run offline CAD harness fixtures | `bun run cad:eval` |
| Run simple offline fixtures | `bun run cad:eval:fast` |
| Run one evidence case | `bun run cad:eval:case <id>` |
| Parse last harness report as JSON | `bun run cad:eval:report` |
| Run real deterministic WASM render benchmark | `bun run cad:eval:render` |

Tests use Bun's built-in test runner. Run `bun run test` before handing off CAD pipeline or skill resolver changes.

The offline evaluator uses `PASS|FAIL|SKIP|ERROR|NOT_RUN` evidence and never marks unexecuted geometry facts as passing. It does not call an LLM or OpenSCAD; treat it as intent/schema/retrieval evidence, not compile, mesh, bbox, semantic-quality, or model-comparison evidence. `cad:eval:render` is a separate real WASM compile/STL/bbox/PNG gate and still does not imply manifold, visual, or model quality. `cad:eval` includes frozen regressions; `cad:eval:fast` is the smallest offline smoke suite.

## Architecture

**AgentSCAD** is a local-first, open-source parametric CAD web app. It has no account system. Users configure their own model provider/API key, submit natural-language descriptions, and receive editable OpenSCAD plus rendered STL/PNG artifacts and validation evidence. Local development stores jobs in SQLite, artifacts on the local filesystem, and custom provider settings in `.agentscad/providers.json`.

OpenSCAD is the core CAD execution engine. Local development can use the native CLI; serverless and explicit `AGENTSCAD_OPENSCAD_BACKEND=wasm` runs use the checksum-pinned official OpenSCAD WASM CLI. LLMs handle request interpretation, SCAD generation, repair, chat, and optional visual review. Deterministic code remains authoritative for compiling/rendering geometry and checking measurable mesh/manufacturing facts.

### Core Pipeline

`src/app/api/jobs/[id]/process/route.ts` is a thin HTTP/SSE adapter. It validates the job exists, checks the state is processable, emits raw SSE frames, and calls `executeCadJob`.

`src/lib/pipeline/execute-cad-job.ts` owns the current runtime state machine:

1. **INTAKE** — analyze the deterministic intent index first; index-unknown requests get one bounded model intake whose untrusted JSON is validated. Any supported ambiguity is persisted and moves to `HUMAN_REVIEW`; the UI records the user's selected meaning before continuing.
2. **GENERATE** — the selected/configured LLM generates structured CAD intent and complete OpenSCAD for both known families and arbitrary freeform requests. Family detection is optional prompt/retrieval context, never a generation gate. Legacy templates for four known families are disabled by default and available only through the explicit `AGENTSCAD_TEMPLATE_FALLBACK=true` demo switch.
3. **RENDER** — native OpenSCAD or the isolated WASM child process renders SCAD to STL; the serverless preview path projects the STL to PNG.
4. **VALIDATE** — deterministic rules inspect compile/render evidence, dimensions, connectivity, holes, wall thickness, and mesh facts. Visual review is a separate user-triggered VLM path.
5. **DELIVER** — SCAD, STL, PNG, parameters, and validation reports are available. `DELIVERED` does not prove that the artifact matches the user's intent or that the user accepted it.

Each step emits SSE events to the requesting frontend. The broader workspace refreshes job data through polling; there is no separate WebSocket service in the current repository.

### Thin Harness, Fat Skills Rules

- Keep CAD reasoning, repair strategy, validation interpretation, and manufacturing judgment in `skills/`.
- Keep deterministic work in code: OpenSCAD rendering, Python/trimesh validation, Prisma writes, artifact paths, SCAD sanitization, SSE formatting, polling adapters, file IO, and tests.
- Preserve runtime contracts unless a migration explicitly updates the frontend and tests: SSE `data: ${JSON.stringify(payload)}\n\n`, existing state strings, existing step strings, `/artifacts/{jobId}/model.stl`, `/artifacts/{jobId}/preview.png`, and the legacy `validationResults` fields `rule_id`, `rule_name`, `level`, `passed`, `is_critical`, `message`. Results may add explicit `status` (`PASS|WARN|FAIL|SKIP|ERROR|NOT_RUN`); skipped/unavailable checks must not be counted as passed.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kevoyuan/AgentSCAD](https://github.com/Kevoyuan/AgentSCAD) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-30 -->

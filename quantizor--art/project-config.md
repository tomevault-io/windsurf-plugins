---
trigger: always_on
description: quantizor's studio: personal creative-coding site on TanStack Start (file-based routes, Nitro, Bun). Ships as a static GitHub Pages tree under `docs/` via `scripts/deploy.sh`.
---

quantizor's studio: personal creative-coding site on TanStack Start (file-based routes, Nitro, Bun). Ships as a static GitHub Pages tree under `docs/` via `scripts/deploy.sh`.

Stack pins (installed)
- TanStack Start / Router: 1.159.4
- React: 19.2.4
- Three.js / @types/three: 0.185.x (WebGPURenderer from `three/webgpu`)
- TypeGPU: 0.9.x (compute only; unused in source until a compute shader lands)
- Tailwind CSS: 4.1.x
- Zod: 3.25.x
- TypeScript: 5.9.x, `strict: true`
- Runtime / tests: Bun

Local carve-outs (override global defaults for this repo)
- No CI. Do not add GitHub Actions or other hosted checks. Local `bun run verify` is the gate.
- No test coverage thresholds or coverage reporting. Do not add a second test runner for branch metrics. Tests accompany behavior changes and must be able to fail for the reason that matters.
- Accessibility work is deprioritized. Do not open WCAG remediation unless the user asks. Keep the `/ui` showcase in sync when design-system APIs change.

Agent directives
- Never run the dev server. Never run `bun dev`, `bun start`, `bun run preview`, or any long-running preview. Assume the app is already being served at `http://art.localhost:3011/` and reuse it. Verification is `bun run typecheck`, `bun test`, and `bun run build` (or `bun run verify`).
- Do not spawn a second Vite/Bun server beside the running one; the port is taken and a stray server splits the logs.
- When planning, include Do's and Don'ts, concrete code samples, and pointers into this file or `agent-docs/`.
- Tracker for deferred work: GitHub Issues. Do not use Changesets (package is `private: true` and unpublished).
- `docs/` is generated Pages output. Never author documentation there. Agent standards live under `agent-docs/`; research notes live under `research/`.
- lightcycle (`src/games/lightcycle/`) has no route. It mounts from `src/components/NotFound.tsx` as the 404 page and participates in the SSR graph that way.

Wayfinding
- This file is the index. Depth lives in `agent-docs/standards/*.md` and `agent-docs/{tanstack-start,react,tailwind-css,typegpu-webgpu,threejs}.md`.
- Design system living docs: `/ui` route plus `src/ui/README.md`, `src/ui/INTEGRATION.md`, `src/ui/TAILWIND-PATTERNS.md`.
- `CLAUDE.md` is `@AGENTS.md` (pointer only).

Prose and communication
- American English. No em-dashes outside quoted material. Plain language over jargon.
- Error messages say what is wrong, where, and what to do. No attribution footers on commits, PRs, or files.
- See agent-docs/standards/prose.md.

Correctness and types
- Types are law. No `any`, no non-null `!`, no `@ts-expect-error` / `@ts-ignore` except as a marked negative test.
- Prefer `unknown` plus type guards. Prefer explicit return types on exported functions.
- Model fixed value sets as `as const` objects plus derived unions, never TypeScript `enum`.
- Zod schemas validate external input and infer types. See agent-docs/standards/types.md.

Testing and validation
- Practice TDD where possible: failing test, then minimal green, then refactor.
- Ship tests with behavior changes. Cover happy paths, empty/null, edges, and failure modes.
- Prefer pure engine logic without Three.js or React. Mock DOM/Three when a component must be tested.
- Use `bun test` for the whole tree (`bun test`), not a path that silently skips suites.
- Coverage is not measured or gated here. Judge a test by whether it can fail for the reason that matters.
- See agent-docs/standards/testing.md.

Inputs and error handling
- Treat the frontend as untrusted. Validate path params, request bodies, `res.json()`, and persisted JSON with Zod or hand-written guards at the boundary.
- Server functions use `.inputValidator` with Zod. Do not use identity validators.
- Worker transferables get discriminant and `byteLength` checks, not Zod.
- See agent-docs/standards/inputs.md.

Organization and naming
- Alphabetize fields in declarations and object literals unless order is load-bearing.
- Persist identifiers in plain English. Prefer reuse of `src/ui/` and shared utils before hand-rolling.
- A source file past about a thousand lines is a prompt to ask whether to split.
- See agent-docs/standards/organization.md.

Comments and docs
- Comments explain why and non-obvious current behavior, never what changed.
- Exported members get hover-friendly docs. Settled designs are documented before implementation.
- See agent-docs/standards/comments.md.

Security and data
- Screen every user-input surface. Do not log cookies, Authorization headers, or other secrets.
- Gate debug logging behind `import.meta.env.DEV` or a tree-shakeable helper so production drops the work.
- See agent-docs/standards/security.md.

Migrations and data lifecycle
- N/A. No database.

Performance
- Validate hot-path claims with a measurement, not recall. Keep drift-prone numbers out of docs; name the command that prints the live figure.
- See agent-docs/standards/performance.md.

Git and process
- Do not commit, push, deploy, or open a PR unless the user asks. Approval is per act.
- Never `git stash`. Prefer a temporary commit when the working tree must be cleared.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [quantizor/art](https://github.com/quantizor/art) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->

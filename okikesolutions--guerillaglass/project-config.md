---
trigger: always_on
description: - **Spec:** `docs/SPEC.md` — source of truth for product requirements, architecture, project format, and verification links.
---

# Repository Guidelines

- **Spec:** `docs/SPEC.md` — source of truth for product requirements, architecture, project format, and verification links.
- **Roadmap:** `docs/ROADMAP.md` — execution sequencing, milestone checklists, and progress tracking.
- **Licensing:** MIT or Apache-2.0 (see spec §19); third-party in `THIRD_PARTY_NOTICES.md`.
- GitHub issues/comments/PR comments: use literal multiline strings or `-F - <<'EOF'` (or `$'...'`) for real newlines; never embed `"\\n"`.

---

## Project Structure & Module Organization

- **Desktop shell (new):** `apps/desktop-electrobun/` — Electrobun main process, React/Tailwind UI, shadcn base components.
- **Desktop shell bridge:** `apps/desktop-electrobun/src/bun/` — Electrobun `BrowserWindow` setup and native engine bridge.
- **Desktop shell UI:** `apps/desktop-electrobun/src/mainview/` — React app (`App.tsx`), UI components, styling.
- **Web app (new):** `apps/web/` — TanStack Start marketing/auth shell with Convex client wiring.
- **Web Convex backend (new):** `apps/web/convex/` — Convex schema/functions for web auth/review/billing surfaces.
- **Protocol (new):** `packages/engine-protocol/` — Zod schemas + TypeScript types for engine requests/responses.
- **Review protocol (new):** `packages/review-protocol/` — Zod schemas + TypeScript types for review bridge requests/events.
- **Schema primitives (new):** `packages/schema-primitives/` — shared Effect Schema helpers reused by protocol packages.
- **Localization (new):** `packages/localization/` — shared locale dictionaries + helpers consumed by renderer UI and desktop shell menus.
- **Native engine (new):** `engines/macos-swift/` — Swift sidecar executable target (`guerillaglass-engine`).
- **Native engine modules:** `engines/macos-swift/modules/` — capture, input tracking, project, automation, rendering, export.
- **Native Windows engine foundation:** `engines/windows-native/` — Rust sidecar foundation with protocol parity handlers.
- **Native Linux engine foundation:** `engines/linux-native/` — Rust sidecar foundation with protocol parity handlers.
- **Native engine shared foundation:** `engines/native-foundation/` — shared Rust runtime/stdio/parity handler implementation consumed by Windows and Linux native sidecars.
- **Rust protocol module (new):** `engines/protocol-rust/` — shared Rust wire message types and capture clock primitives.
- **Platform stub engines:** `engines/windows-stub/`, `engines/linux-stub/` — protocol-compatible stubs for parallel engine work.
- **Swift protocol module (new):** `engines/protocol-swift/` — wire codec and typed message envelope models.
- **Tests:** `Tests/` — `automationTests/`, `captureTests/`, `engineProtocolTests/`, `exportTests/`, `projectMigrationTests/`, `renderingDeterminismTests/`.
- **Gate scripts (implementation):** `Scripts/` — `rust_gate.sh`, `typescript_gate.sh`, `full_gate.sh`, `docs_gate.mjs`, `coverage.sh`, `rust_coverage.sh`, `swift_coverage.sh`, `coverage_check.sh`.
- **Docs:** `docs/` — SPEC, ROADMAP, and other project docs.

When adding modules or moving code, keep the spec’s architecture (§16–17) and update `AGENTS.md` / `docs/SPEC.md` / `docs/ROADMAP.md` if the tree or planned milestones change.

---

## Build, Test, and Development Commands

- **Rust gate (fmt, lint, test):** `bun run gate:rust`  
  Runs: `cargo fmt --all --check` → `cargo clippy --workspace --all-targets -- -D warnings` → `cargo test --workspace --all-targets`.
- **JS/TS format check:** `bun run js:format:check`  
  Runs `oxfmt --check` on workspace JS/TS code.
- **JS/TS lint:** `bun run js:lint`  
  Runs `bunx oxlint .`; project rules and type-aware settings are auto-discovered from `oxlint.config.mjs`.
- **React effect guard lint:** `bun run js:lint:react-effects`  
  Runs the custom guard that rejects direct React state updates in effects (`Scripts/lint_no_state_updates_in_effect.mjs`).
- **TypeScript gate (format + lint + typecheck + tests):** `bun run gate:typescript`  
  Runs: JS/TS format check (Oxfmt) → JS/TS lint (Oxlint) → React effect guard lint → docs coverage gate (TypeScript surfaces) → desktop shell typecheck → landing app typecheck → protocol package typecheck → desktop tests.
- **Docs coverage gate (all surfaces):** `bun run docs:check`  
  Runs coverage checks for TypeScript, Swift, and Rust public/exported API surfaces using `docs/doc_coverage_policy.json`.
- **Docs coverage gate (TypeScript surfaces):** `bun run docs:check:ts`
- **Docs coverage gate (native surfaces):** `bun run docs:check:native`
- **Full gate (rust + typescript + swift):** `bun run gate`  
  Runs: `gate:rust` → `gate:typescript` → docs coverage gate (Swift/Rust surfaces) → SwiftFormat → SwiftLint → `swift test` → `swift build`. Use this to verify the project after changes.
- **Desktop deps (workspace):** `bun install`
- **Desktop shell dev:** `bun run desktop:dev`
- **Desktop shell dev fallback (macOS app open):** `bun run desktop:dev:open`
- **Desktop shell dev with HMR:** `bun run desktop:dev:hmr`
- **Web app dev (TanStack Start + Convex):** `bun run web:dev`
- **Web app build:** `bun run web:build`
- **Web app typecheck:** `bun run web:typecheck`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [okikeSolutions/guerillaglass](https://github.com/okikeSolutions/guerillaglass) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->

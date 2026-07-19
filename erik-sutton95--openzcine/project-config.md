---
trigger: always_on
description: Open-source iOS + Android app to connect to, control, and monitor (live image) Nikon Z
---

# OpenZCine

Open-source iOS + Android app to connect to, control, and monitor (live image) Nikon Z
cinema-line cameras — primarily the **Nikon ZR**. Built openly with agentic coding tools, held to
clean, exemplary engineering standards.

> **Status:** Native iOS and Android implementation milestone. Production work targets the portable
> Swift business/protocol core, a SwiftUI iOS shell, and a Jetpack Compose Android shell. Android is
> not publicly released yet. The older Flutter prototype remains only as a protocol/live-view
> reference while useful.

## Stack & tooling

- **Swift Package Manager / Swift** — production shared business and camera-protocol core.
- **SwiftUI** — production iOS app shell.
- **Jetpack Compose / Kotlin** — production Android app shell and Android platform adapters.
- **Swift SDK for Android** — production bridge path for the shared Swift core. Keep the facade/JNI
  boundary small and verify build, packaging, and runtime behavior through the Android gates.
- **Flutter / Dart** — archived prototype reference only; not part of production tooling.
- **just** — the single entry point for all repository tasks. Run `just` to list recipes.
- **swift-format / swift test / xcodebuild** — production formatting, tests, and iOS build checks.
- **typos, editorconfig-checker, markdownlint-cli2, lychee, actionlint** — repository meta-checks.

Install local tooling with `just setup` (macOS / Homebrew).

## Where things live

- `Sources/OpenZCineCore/` — production Swift shared core.
- `Tests/OpenZCineCoreTests/` — Swift shared-core tests.
- `ios/Runner/` — production SwiftUI iOS app shell.
- `Apps/Android/` — production Jetpack Compose app, platform adapters, and Wear OS companion.
- `reference/flutter-prototype/` — archived Flutter prototype/reference source.
- `vendor/` — **gitignored.** Local-only material; nothing here is committed or required to build.
- `ref/` — **gitignored** local reference material.
- `docs/` — engineering references: `commit-hygiene.md` (what must never be committed),
  `nikon-mtp.md` (protocol sourcing and maintenance policy), `nikon-sdk.md` (no-vendor-SDK policy),
  `PROJECT-MANAGEMENT.md` (Kaneo board conventions + agent sync contract).
- `docs/design/` — design specs, implementation plans, and archived browser prototypes.
- `docs/investigations/` — resolved or pending engineering investigations and debugging records.
- `site/` — deploy-ready GitHub Pages landing page; no raw design sources.
- `.local/` — **gitignored** demo feeds, raw marketing sources, and local tooling migrations.
- `.github/` — CI workflows, issue/PR templates.

## Supported agent tools

OpenZCine supports **Claude Code** and **Codex** only:

- Shared instructions live in `AGENTS.md`; both clients follow them.
- Claude Code configuration lives under tracked `.claude/`, with personal settings ignored.
- Codex uses `AGENTS.md`; personal `.codex/` state is ignored.

Do not add Cursor, Copilot, Gemini, Windsurf, Aider, Cline, Roo, Kilo, Continue, or OpenCode-specific
instructions. Put guidance needed by both supported clients here or in the relevant project docs.

For `docs/flows/` work, read its `README.md` first, pull the live ExcaliDash scene before editing,
use `pushMerged` for incremental changes, and verify afterward. `just flows-push` is reserved for an
intentional whole-scene replacement because it resets the live layout.

## Hard rules

- **Never commit `vendor/`, `ref/`, `captures/`, or `.local/`.** They contain proprietary,
  private, or raw working material. They are gitignored; if a `git add` ever tries to include them,
  fix `.gitignore` — do not force-add.
- **Mind commit hygiene.** This is a public repo; never commit secrets, credentials, PII, or
  non-redistributable assets. Follow `docs/commit-hygiene.md` before every commit.
- **Work on a branch and open a PR.** Do not commit directly to `main`.
- **Conventional Commits** for every commit (`feat:`, `fix:`, `docs:`, `chore:`, `ci:`, `build:`, `test:`).

## Verification

Before claiming any change is complete, run:

```sh
just check
```

For production Swift/iOS or Swift Android-facade changes, also run:

```sh
just native-check
```

## Project management (Kaneo)

Work is tracked on the OpenZCine Kaneo board (project `OPE`). Keep it in sync as part of normal
work — this is a standing contract, not a request-driven step:

- **Start** a deliverable → set its task to **in-progress**.
- **Open/update its PR** → move it to **in-review** through review, even after local Verification.
- **Finish** it → set it to **done** only after merge and successful default-branch CI.
- **Abandon** it → move it to **archived** explicitly; never infer cancellation from a closed PR.
- **New scope** → create a task (backlog ideas go to **planned** with the `investigation` label);
  add real deliverables to `docs/ROADMAP.md`.
- Roadmap = scope source of truth; Kaneo = live status mirror. Move them together.

Board identity, statuses, the GitHub issue sync behavior, and which MCP tools work on this
self-hosted instance are in `docs/PROJECT-MANAGEMENT.md`. Read it before touching the board.

## Coding principles

- **Composition over inheritance.** Prefer small, pure functions and immutable data over stateful

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [erik-sutton95/OpenZCine](https://github.com/erik-sutton95/OpenZCine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-18 -->

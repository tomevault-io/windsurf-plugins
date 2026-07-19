---
trigger: always_on
description: This file provides unified guidance for all AI coding agents (Claude Code, Codex, etc.) working in this repository.
---

# AGENTS.md

This file provides unified guidance for all AI coding agents (Claude Code, Codex, etc.) working in this repository.

## Repository Facts

- Agentre is a Wails v2 desktop app: Go 1.26 backend + React 19 / TypeScript frontend.
- Main tech stack: Go 1.26, Wails v2, React 19, TypeScript, Vite, Tailwind CSS v4, pnpm 10.33.
- The Go module path is `github.com/agentre-ai/agentre`.
- Frontend-backend IPC only goes through the Wails bindings in `internal/app`; the generated bindings live in `frontend/wailsjs`; **do not add HTTP-style app APIs**.

This repository produces two binaries:

- **`agentre`** (root `main.go`) — the desktop app. It also doubles as a CLI shim: `agentre claudecode …` short-circuits to `internal/cli/claudecodecmd` before booting wails/cago (used by Claude Code hook subprocesses).
- **`agentred`** (`cmd/agentred/`) — a headless daemon that executes claude-code / codex subprocesses on behalf of a paired desktop over JSON-RPC-over-WebSocket on the LAN. The daemon-side handlers live in `internal/daemon/`.

## High-Priority Constraints (mandatory, non-negotiable)

The following are hard rules. If the current task conflicts with them, **stop and ask the user first** — do not work around them on your own.

1. **Strict TDD / BDD: Red → Green → Refactor, no exceptions.**
   - For a new feature, first write a BDD-style behavior spec (`Given … When … Then …` or goconvey `Convey("when X, then Y")`) covering the happy path plus at least one boundary/error case, **then write the implementation**.
   - Do not write implementation code without a failing test. See [docs/development.md](docs/development.md) for details.
2. **Verify the bug exists before fixing it.**
   - Write a regression test that reproduces the failure, **run it and watch it fail** (and fail for the right reason), then start patching.
   - If the bug genuinely cannot be reproduced in a test, **tell the user explicitly**, and then discuss the patch approach. Do not silently "this is probably how to fix it" and change code.
3. **Prefer refactoring over patching — fix the root cause, don't mask it.**
   - Fix the bad value the producer emits, instead of adding an `if x == nil` fallback guard at every consumer; don't repeatedly normalize the same field at multiple call sites — normalize once at the boundary.
   - A comment like `// workaround because X returns Y` is a smell; the code underneath most likely needs to change. Refactor bad structure away when you can rather than piling on patches — but keep the refactor **within the scope of the current task** and don't let it spill over.
4. **Do not modify files unrelated to the current task.**
   - The diff should only touch the producer + its tests, plus at most one obvious in-scope drift.
   - **No** drive-by refactor / rename sweep / formatter pass / dead-code cleanup / import reordering / unrelated test churn — they bury the real change and break `git bisect`.
   - When you see unrelated dirty data, flag it to the user and ask, **do not fix it in passing**.
5. **New visible frontend UI copy must go through i18n.**
   - New UI text uses `react-i18next`'s `t(...)`, and updates both `frontend/src/i18n/locales/zh-CN/common.json` and `frontend/src/i18n/locales/en/common.json`.
   - Do not add hardcoded Chinese; ESLint, via `eslint-plugin-i18next`'s `i18next/no-literal-string`, blocks hardcoded Chinese UI copy in JSX text and visible attributes.
   - Static `t("...")` keys and locale coverage are validated by `frontend/src/__tests__/i18n.test.ts`; run the relevant tests when you change copy.
   - Do not translate dynamic output such as agent / user / terminal / code / markdown; it naturally never enters `t(...)`, and using a global text-rewrite fallback is forbidden.
   - All static UI copy is explicitly `t(...)`. See [docs/frontend.md](docs/frontend.md) for details.

## SOLID (coding rules)

Run every new package / type / function through this before merging:

- **S — Single Responsibility (SRP).** One reason to change is enough. If a service does parsing + persistence + notification at once, split it; a function that runs hundreds of lines almost certainly violates SRP.
- **O — Open/Closed (OCP).** Extend by adding a new type / strategy, not by editing a switch scattered across multiple callers. A new agent backend (Claude Code / Codex / built-in) should be an interface implementation, not a patch to `if engine == ...`.
- **L — Liskov Substitution (LSP).** An implementation must hold the interface contract (nil semantics, error types, side-effect boundaries); there should be no surprise like "this implementation ignores ctx".
- **I — Interface Segregation (ISP).** Define interfaces on the **consumer** side and narrow them as needed: a service declares only the few repo methods it uses, instead of making the consumer depend on a fat 20-method interface.
- **D — Dependency Inversion (DIP).** A `service` depends on a repository **interface**, never on the concrete implementation; the concrete implementation is wired up in `main.go` / `internal/app` via `RegisterXxx(impl)` — which is exactly what makes TDD mocking possible.

## High cohesion, low coupling (coding rules)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agentre-hub/agentre](https://github.com/agentre-hub/agentre) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-18 -->

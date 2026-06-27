---
trigger: always_on
description: KKTerm is a cross-platform, local-first Tauri v2 desktop app: Rust backend,
---

# Agent Instructions

## Project Shape

KKTerm is a cross-platform, local-first Tauri v2 desktop app: Rust backend,
React/TypeScript frontend, SQLite for non-secret durable data, and OS keychain
for secrets.

Use these docs as the source of truth instead of copying their rules here:

- `CONTEXT.md` — product vocabulary and domain boundaries.
- `docs/ARCHITECTURE.md` — architecture, source map, command boundaries,
  native-window rules, Settings rules, overlays, i18n, and UI placement.
- `docs/PRD.md` / `docs/ROADMAP.md` — product scope and direction.
- `docs/ADR/` — accepted architectural decisions.
- `docs/manual/INDEX.md` — shipped operation manual and chapter index.

Before changing behavior, terminology, or source placement, read the relevant
source-of-truth docs and preserve their boundaries.

## Constitution

These rules apply to every task unless explicitly overridden.
Bias: caution over speed on non-trivial work. Use judgment on trivial tasks.

## Rule 1 — Think Before Coding

State assumptions explicitly. If uncertain, ask rather than guess.
Present multiple interpretations when ambiguity exists.
Push back when a simpler approach exists.
Stop when confused. Name what's unclear.

## Rule 2 — Simplicity First

Minimum code that solves the problem. Nothing speculative.
No features beyond what was asked. No abstractions for single-use code.
Test: would a senior engineer say this is overcomplicated? If yes, simplify.

## Rule 3 — Surgical Changes

Touch only what you must. Clean up only your own mess.
Don't "improve" adjacent code, comments, or formatting.
Don't refactor what isn't broken. Match existing style.
`cargo fmt` is optional in this repo. Rust formatting must use the Rust 2024
edition, as configured by `rustfmt.toml`. If formatting is useful, run it only
on the smallest practical Rust scope you intentionally touched, such as a
single file or single Rust source file. Do not run broad `cargo fmt` over the
whole workspace unless the user explicitly asks for global formatting; it can
rewrite unrelated Rust sources and create noisy file-format churn.

## Rule 4 — Goal-Driven Execution

Define success criteria. Loop until verified.
Don't follow steps. Define success and iterate.
Strong success criteria let you loop independently.

## Working Rules

- Domain language lives in `CONTEXT.md`. Use **Connection** for durable openable
  resources, **Session** for live runtime state, and **Tab** for the frontend
  workspace container. Do not say "profile" for a stored Connection.
- Capital-M **Module** means a top-level Activity Rail destination. Current
  product Modules are **Workspace**, **Dashboard**, and **Install Helper**;
  **Settings** is the bottom rail destination. The App Launcher is a Dashboard
  widget, not a Module.
- Source placement lives in `docs/ARCHITECTURE.md` → "Frontend Source Map".
  Prefer existing source areas and typed wrappers in `src/lib/tauri.ts`.
- The operation manual ships with the app. Any UI behavior change in a manual
  chapter's scope must update the relevant chapter in `docs/manual/`; reference
  i18n keys, not English labels.
- Tutorial-capable UI needs a stable `data-tutorial-id`, a navigation entry in
  `src/app/tutorialNavigationModel.ts`, matching `tutorial_highlight` metadata
  in `src-tauri/src/ai.rs`, and manual grep hints. `npm run check` validates
  these mappings.
- All user-visible strings go through i18n. Add English keys first in
  `src/i18n/locales/en.json`; whenever new UI strings are created or changed,
  follow `docs/localization_todo/README.md` exactly. If translations are not
  completed in the same change, add one pending file per key under
  `docs/localization_todo/` using that README's flow and template. When the
  meaning of an English word shifts by context (e.g. "Play" starts media, runs
  something, or names a theatrical play — each translates differently), create a
  separate key per context instead of reusing one; reuse a key only when the
  meaning is identical everywhere. Keep interpolation placeholders
  translation-safe: use named `{{…}}` placeholders, one full sentence per key,
  no concatenated fragments around a variable. See
  `docs/manual/16-localization.md` and `docs/ARCHITECTURE.md`.
- **zh-TW must never contain Mainland Chinese terminology.** This is a critical
  review gate. `zh-TW.json` uses Taiwan computing terminology — never Mainland
  terms, even in traditional characters. See the forbidden→required term mapping
  in `docs/manual/16-localization.md` (e.g. 連線 not 連接, 終端機 not 終端,
  儲存 not 保存, 預設 not 默認, 資料 not 數據, 伺服器 not 服務器, 客戶端 not
  用戶端, 使用者 not 用戶, 程式 not 程序, 螢幕 not 屏幕, 萬用字元 not 通配符,
  介面 not 接口). Never copy from `zh-CN.json` and convert characters.
- The UI design language (tokens, dialog primitives, the `ConfirmSheet`
  confirmation template, button order, and the SFTP file-browser pattern) lives
  in `docs/DESIGN_LANGUAGE.md`. Read it before adding any dialog, sheet, settings
  surface, or file-browser UI. Build dialogs from `src/app/ui/dialog` primitives
  and read color tokens from `src/styles/colorSchemes.css`; never hard-code hex.
- App-owned popup dialogs use a single concise title by default. Do not add a
  subtitle or explanatory header copy unless the flow truly needs it; put

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ryantsai/KKTerm](https://github.com/ryantsai/KKTerm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->

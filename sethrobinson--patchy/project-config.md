---
trigger: always_on
description: This is repository-wide agent policy. Read it at the start of each task before inspecting files, running commands, or planning work. Reread it only if the repository changes, this file changes, or its contents are no longer available in context.
---

# Repository Instructions

This is repository-wide agent policy. Read it at the start of each task before inspecting files, running commands, or planning work. Reread it only if the repository changes, this file changes, or its contents are no longer available in context.

If `agents_local.md` exists at the repository root, open and read it too. It holds knowledge specific to this checkout and developer, such as which systems it can ssh into and use for building. Anything sensitive goes in `agents_secret.md` beside it, read only if it exists and only when a task needs it. Both are optional and gitignored, and must never be checked in. When neither exists, carry on without them; their absence is not an error. This repository is public, so machine names, users, addresses, local paths, and credentials belong in those files, never in tracked files. A git worktree has no untracked files: look for them in the main checkout, the parent of `git rev-parse --path-format=absolute --git-common-dir`.

Keep this file at or below 30,000 bytes. Detailed implementation knowledge belongs in `docs/<topic>.md`; read the relevant linked document before working in that area, update it when behavior changes, and do not duplicate its details here. Every file under `docs/` must also stay at or below 30,000 bytes (Seth, August 2026): keep docs dense and current-state only. Cut narrative history, experiment logs, and restatements of constants that live in code; never cut normative rules, calibration facts recorded only in the doc, or headings cited from code comments.

## Repository-wide rules

- Every PSD/PSB Patchy writes, including script and MCP output, must open in Adobe Photoshop without warnings or errors. Custom metadata is allowed only when Photoshop accepts the file without warning, repair, or data-discard prompts. Follow the compatibility contract in [docs/ps-compat.md](docs/ps-compat.md).
- When adding or changing user-facing text, make it extractable (`tr()`, a literal-context `translate`, `QT_TR_NOOP`/`QT_TRANSLATE_NOOP` for bound text, `PATCHY_TRANSLATE_NOOP` in Qt-free code), run `scripts\update-translations.ps1`, and fill the new entries in every `translations/patchy_<code>.ts` in the same change. Never hand-edit catalog structure. The catalog tests fail otherwise. See [docs/localization.md](docs/localization.md).
- Tests that need files outside the project must first copy them into `local-test-fixtures`; never add hardcoded external paths such as `C:\temp` or `D:\projects` to test code.
- Commit automatically only after a finished piece of work is verified and its required handoff is complete. Do not commit failing or half-finished states. Never push unless Seth explicitly asks in the current request.
- Never add AI attribution, generated-with text, or an OpenAI/Codex/Claude co-author to commits or pull requests. Keep commit messages to a concise subject and at most one short supporting line.
- Driving Adobe Photoshop through COM (`New-Object -ComObject Photoshop.Application`, `DoJavaScript`, Action Manager) is ALWAYS authorized for capture, verification and acceptance work; no per-request permission is needed (Seth, September 2026). What stays forbidden without explicit authorization in the current request is anything that sees or controls the desktop itself: screenshots of the desktop (it may show sensitive data), Computer Use, desktop UI automation, SendInput, clicking and typing. Use Patchy's command-line screenshot and automation surfaces where possible; see [docs/testing.md](docs/testing.md).
- Build/test housekeeping is already authorized and does not require another confirmation: create, rename, replace, or delete generated build artifacts, temporary executable backups, test-owned socket files, logs, and scratch files inside this repository's build or test-output directories after verifying the exact paths and ownership. This includes an obsolete executable preserved under a temporary name during a rebuild. Stopping processes launched for the current build/test run is also authorized. This permission does not cover user documents, fixtures, source files, unrelated files/processes, or a running user app/connector; those retain the rules above.
- Supported Windows, macOS, and Linux Debug and Release builds must have zero compiler, linker, and `lrelease` warnings. Keep warnings non-fatal. Fix Patchy-owned code explicitly. For vendored sources compiled into Patchy-owned targets, scope a suppression to one source and diagnostic. See [docs/platform.md](docs/platform.md).
- User-facing documentation must not use em dashes. Write plain, direct prose without hype, emoji headings, "not just X, but Y" constructions, or stock AI phrasing such as "seamlessly", "robust", "comprehensive", and "delve".

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SethRobinson/Patchy](https://github.com/SethRobinson/Patchy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->

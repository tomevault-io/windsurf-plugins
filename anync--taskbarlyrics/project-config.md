---
trigger: always_on
description: Read this file before changing the repository. Keep user-visible behavior and stored user data stable unless the user explicitly approves a migration.
---

# TaskbarLyrics repository instructions

Read this file before changing the repository. Keep user-visible behavior and stored user data stable unless the user explicitly approves a migration.

## Mandatory Clean Code workflow

- For every code/configuration plan, implementation, bug fix, refactor, code review, or verification task, read `.agents/skills/taskbarlyrics-clean-code-guardian/SKILL.md` completely before acting.
- Read every reference that skill marks as required. This applies even when the skill is not shown in the active skill catalog.
- Documentation-only copy edits do not require the skill unless they change development policy, architecture, behavior contracts, or verification instructions.

## Build and verification

- Requires .NET 8 SDK, Windows x64, and Windows 11 SDK 10.0.22621.
- Run: `dotnet run --project TaskbarLyrics.App`
- Build: `dotnet build TaskbarLyrics.sln`
- Targeted verification while iterating: `powershell -ExecutionPolicy Bypass -File scripts/verify.ps1 -Tier Targeted -Area Core -Filter FullyQualifiedName~TestClassName`
- Affected-project verification: `powershell -ExecutionPolicy Bypass -File scripts/verify.ps1 -Tier Project -Area Core` (areas: `Core`, `App`, `Web`, `Settings`; multiple areas are allowed)
- Full delivery verification: `powershell -ExecutionPolicy Bypass -File scripts/verify.ps1`
- Stop local app before a solution build: `powershell -ExecutionPolicy Bypass -File scripts/restart-app.ps1 -StopOnly`
- Restart local app after delivery: `powershell -ExecutionPolicy Bypass -File scripts/restart-app.ps1 -NoWait`
- Release packaging: `powershell -ExecutionPolicy Bypass -File scripts/publish-release.ps1`
- Before handoff, also run `git diff --check`. For production code changes, require a zero-warning solution build.
- After verification succeeds for a change that affects the runnable app, run `scripts/restart-app.ps1 -NoWait` and leave the app ready for user validation. Skip this for documentation-only, test-only, instruction-only, or build-only changes, or when the user opts out.

### Delivery execution order

- Before the first test or full verification, apply whitespace formatting once to every changed C# file. This catches CRLF and basic formatting drift before expensive verification.
- Use targeted verification while iterating. At the delivery boundary, run full verification once after the implementation is complete.
- Before the required zero-warning solution build, stop the running local app with `restart-app.ps1 -StopOnly`; it otherwise locks App output assemblies on Windows.
- After a successful solution build and `git diff --check`, start the validated app with `restart-app.ps1 -NoWait`. The default restart command remains available for interactive foreground development.

The verification script defaults to `Full`, which runs Vitest/jsdom web tests, App tests, Core tests, the settings contract test, and `dotnet format --verify-no-changes`. Use `Targeted` for the directly affected test class or web test file while iterating, `Project` after the affected feature is complete, and `Full` only at the delivery boundary. Tests change when an observable behavior or compatibility contract changes; implementation-only refactors should preserve existing tests whenever practical.

### Verification output discipline

- Keep complete verification stdout and stderr in ignored logs under `tmp/verify-logs/`; do not stream successful per-test output or paste raw logs into agent handoffs.
- On success, report only the command or verification tier, affected area, pass/fail summary, duration when available, and log path. On failure, report the failing step, exit code or exception, a bounded excerpt containing the actionable failure evidence, and the full log path.
- Expand detailed output only for a failed or ambiguous step. Prefer a targeted rerun over loading an entire full-verification log into model context.
- A Luna handoff must summarize verification evidence rather than copy logs. Sol should not rerun the same targeted verification when Luna's result and log are current and sufficient; Sol still owns required integration and full delivery verification.
- Output reduction must never hide warnings, failures, skipped checks, or an unexecuted command. Preserve the original nonzero failure semantics and keep the complete log available for diagnosis.

## Solution layout

- `TaskbarLyrics.Core`: platform-neutral lyric retrieval, matching, parsing, caching, local media indexing, persistence, and policies. Do not add WPF, WebView2, WinForms, or native-window dependencies here.
- `TaskbarLyrics.App`: Windows host using WPF, WinForms NotifyIcon, WebView2, SMTC, audio capture, native-window integration, and the composition root.
- `TaskbarLyrics.Core.Tests` and `TaskbarLyrics.App.Tests`: xUnit regression tests.
- `tests/web`: Vitest/jsdom behavior tests.
- `TaskbarLyrics.App/Web`: modular lyrics and settings interfaces hosted in WebView2.

## Behavior contracts

- Preserve existing `settings.json` fields and migration behavior unless a migration is explicitly requested and tested.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ANYNC/TaskbarLyrics](https://github.com/ANYNC/TaskbarLyrics) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->

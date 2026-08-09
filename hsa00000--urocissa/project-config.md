---
trigger: always_on
description: This file applies to the entire repository. The frontend rules below are mandatory for
---

# Urocissa Agent Instructions

This file applies to the entire repository. The frontend rules below are mandatory for
`gallery-frontend` and for any other Vue/Vuetify interface added to this repository.

## Command execution and local service safety

These rules apply to all repository work, including investigation, testing, benchmarks,
and frontend or backend development.

### Package manager and validation commands

- `gallery-frontend` is an npm project because it has `package-lock.json`. Use npm
  scripts or the already-installed local binaries; do not use pnpm, Yarn, or another
  package manager against its `node_modules`.
- Do not let a fallback runner install, move, or repair dependencies implicitly. Any
  dependency installation or network access must be intentional and reported.
- Run checks that share the same `node_modules` sequentially. Do not launch TypeScript,
  Vitest, ESLint, and build commands concurrently through package-manager processes.

### Expensive hybrid virtual-scroll validation is opt-in

- The hybrid virtual-scroll browser gate is a manual, expensive diagnostic. Do not run
  `performance/run-hybrid-scroll-gate.ps1` with either `Quick` or `Full` unless the user
  explicitly asks to run the hybrid-scroll tests, scroll performance/visual validation, or an
  equivalent complete browser gate in the current request.
- The same opt-in rule applies to direct equivalents of that gate: repeated hybrid-scroll browser
  scenarios, Windows trusted wheel/touch injection, compositor elastic-overscroll captures,
  performance traces, and the desktop/mobile visual matrix. A code change touching virtual
  scrolling does not by itself authorize any of these expensive checks.
- Without an explicit request, use only proportionate fast validation, such as `cargo test`, a
  targeted Vitest file, a focused type check, or another ordinary project check appropriate to the
  files changed. In the handoff, state that the hybrid-scroll gate was not run because it was not
  explicitly requested; this is expected and is not a validation failure.
- When the user does explicitly request this validation, select and run `Quick` or `Full` according
  to `docs/HYBRID_VIRTUAL_SCROLL_TESTING.md`. This opt-in rule takes precedence over generic
  benchmark, visual-check, pre-merge, and release checklists for this specific gate and its direct
  equivalents.

### Long-running processes

- Check the expected port or health endpoint before starting a local service and reuse
  an already-running service when it is the intended instance.
- Treat Vite, backend servers, watchers, and similar programs as managed services, not
  ordinary commands that are expected to exit. Retain their PID or managed cell/session
  identifier, verify readiness separately, and stop processes started for the task
  during cleanup.
- Do not launch a long-lived process through PowerShell `Start-Process` while also using
  `-RedirectStandardOutput` or `-RedirectStandardError`; this combination can prevent
  the command runner from returning. Prefer a managed yielded process/session or a
  dedicated terminal.
- A service launch must produce an identifier and pass its readiness check promptly.
  If it does not, stop and inspect the process instead of retrying the same launch
  pattern or waiting indefinitely.

### Errors, timeouts, and waiting

- For PowerShell command blocks, set `$ErrorActionPreference = 'Stop'`. Check
  `$LASTEXITCODE` after native commands when their exit status matters so a
  non-terminating PowerShell error cannot be mistaken for success.
- Set an internal deadline for polling, browser navigation, benchmarks, and network
  requests. The outer command timeout must be longer than the command's worst-case
  internal deadline.
- Use short, bounded wait intervals. After two consecutive waits with no new output,
  inspect the process, port, or logs, or terminate it; do not continue blind waiting.
- Use `rg` with explicit exclusions for repository searches. Avoid unrestricted
  recursive filesystem scans that traverse `node_modules`, build outputs, or large
  generated directories.
- During benchmark development, run one sample or the smallest relevant scenario
  first. Run the full repeated-sample gate only after the command is known to complete
  reliably, and surface per-sample progress for longer runs.

## Vuetify-first frontend policy

All frontend work must preserve a consistent Vuetify visual language. Prefer Vuetify's
components, props, slots, theme system, display helpers, and utility classes over native
HTML layout wrappers or hand-written visual systems.

### 1. Use Vuetify components before native HTML

- Use the closest Vuetify component whenever one exists:
  - Layout: `VContainer`, `VRow`, `VCol`, `VSheet`, `VSpacer`.
  - Surfaces: `VCard`, `VCardItem`, `VCardText`, `VCardActions`, `VToolbar`.
  - Lists and grouped settings: `VList`, `VListItem`, `VListSubheader`, `VDivider`.
  - Controls: `VBtn`, `VCheckbox`, `VCheckboxBtn`, `VSwitch`, `VTextField`,
    `VSelect`, `VAutocomplete`, `VCombobox`, and other Vuetify inputs.
  - Feedback: `VAlert`, `VProgressLinear`, `VProgressCircular`, `VEmptyState`,
    `VSnackbar`, `VTooltip`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hsa00000/Urocissa](https://github.com/hsa00000/Urocissa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->

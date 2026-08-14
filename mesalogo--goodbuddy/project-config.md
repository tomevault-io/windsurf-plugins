---
trigger: always_on
description: These instructions apply to the entire repository.
---

# GoodBuddy Agent Guide

## Scope

These instructions apply to the entire repository.

GoodBuddy is a secure, cross-platform Electron desktop assistant. It uses
Electron, React, TypeScript, Vite, Vitest, and SQLite. User-facing copy is
primarily Simplified Chinese.

## Architecture

- `src/main`: privileged Electron main process, runtimes, persistence, IPC,
  knowledge, automation, and OS integration.
- `src/preload`: the narrow, typed bridge exposed to the renderer.
- `src/shared`: schemas, contracts, presets, and IPC channel definitions shared
  across process boundaries.
- `resources/skills`: bundled skills.
- `build`: packaging scripts and icons.
- `out` and `dist`: generated output. Change source files instead.

Keep Electron security boundaries intact:

- Preserve context isolation and sandboxing.
- Never enable renderer Node integration.
- Validate IPC input with shared Zod schemas and verify trusted senders.
- Expose only explicit preload methods. Do not pass raw Electron APIs.
- Keep API keys in the main process and encrypted settings store.

## Runtime Behavior

- Ask mode must remain read-only at the runtime boundary.
- Execute mode may use tools only through the existing approval controls.
- Preserve cancellation, timeout, bounded-output, and shutdown behavior.
- Treat OpenCode and Continue as untrusted child runtimes. Preserve environment
  allowlists, sandbox checks, and per-tool approval enforcement.
- A successful image-model configuration check does not prove generation works.
  Only an actual generation request verifies the provider path.
- Do not fetch provider-returned image URLs. Accept and validate bounded inline
  image data, then persist it as an artifact in the main process.

## Data and Compatibility

- Preserve existing user data and migrations.
- Do not weaken SQLite transaction, lifecycle, deduplication, or cleanup logic.
- Treat user workspaces and untracked files as user-owned.
- Keep Windows, macOS, Linux x64, and Linux arm64 behavior in mind.
- Do not hard-code machine-specific paths, credentials, or provider endpoints.

## Implementation Conventions

- Follow surrounding TypeScript and React patterns.
- Reuse installed libraries and shared contracts before adding dependencies.
- Keep changes focused. Do not add unrelated refactors or documentation.
- Add or update focused tests for behavioral changes and regressions.
- Avoid broad catches that erase HTTP status, cancellation, or provider error
  context.
- Keep UI accessible with labels, keyboard behavior, semantic roles, and visible
  focus states.

## UI Consistency

- Treat `UI-DESIGN.md` as the canonical UI design system. Read and follow it
  before changing renderer layout, shared controls, interaction feedback,
  themes, responsive behavior, or accessibility semantics.
- Reuse the shared `PageTabs` and `SegmentedControl` primitives instead of
  creating page-specific tab or toggle styles. A semantic tab set may use the
  shared segmented visual variant, but it must retain `tablist`, `tab`,
  `tabpanel`, `aria-selected`, roving focus, and arrow-key behavior.
- Use the shared sliding Switch pattern for persistent binary states and expose
  `role="switch"` even when it is implemented with a checkbox input. Keep
  Checkbox visuals and semantics for multi-select, assignment, and explicit
  confirmation. Do not create page-specific Switch styling.
- Use the bundled `Inter Variable` and `Noto Sans SC Variable` UI fonts through
  the shared typography tokens. Do not add remote font requests or page-local
  font stacks. Keep redistributed font licenses in packaged resources and
  retain system fallbacks for startup and unsupported glyphs.
- Route transient success and informational feedback, plus asynchronous errors
  that are not tied to one field, through the application notification
  viewport. Do not render page-local copies of the same notification pattern.
- Keep inline feedback only when it must remain attached to its context, such
  as field validation, destructive confirmation, operation progress, a
  blocking page state, or an error with an immediate local recovery action.
- Do not show the same event both inline and as an application notification.
  Preserve user input and actionable error context when an operation fails.

## Release Packaging

- `.github/workflows/packages.yml` is the canonical cross-platform packaging
  workflow. It validates and builds `out` once, then packages on six native
  runners: Windows, macOS, and Linux, each for x64 and arm64.
- Run the unified packager with
  `npm run release:package -- --platform <platform> --arch <arch>`. It only
  packages for the native host and writes to
  `dist/release/<platform>-<arch>`.
- Default deliverables are NSIS and portable ZIP for Windows, DMG and ZIP for
  macOS, and AppImage and DEB for Linux. Every target includes
  `release-manifest.json` with SHA-256 hashes.
- `build/build-release.cjs` verifies the unpacked application, `app.asar`,
  bundled Continue and OpenCode runtimes, executable architecture, and package
  signatures before atomically replacing a release directory.
- Keep electron-builder invocations on `--publish never`. Main-branch builds
  run validation and build the production bundle without running the native

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mesalogo/goodbuddy](https://github.com/mesalogo/goodbuddy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->

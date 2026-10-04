---
trigger: always_on
description: > **General working rules are vendored, not duplicated here:** see
---

# AI Assistant Instructions

> **General working rules are vendored, not duplicated here:** see
> `.github/instructions/working-principles.instructions.md` (upstream:
> `fuzzifikation/agents`, re-sync with `G:\agents\bin\sync.ps1`). Git/push
> discipline, version and release law, changelog epistemology, review
> governance, verification laws, simplicity laws and communication style live
> there and apply to every repo. **Everything below is this project only.**

---

## This Repository: vLLM-Copilot

### Architecture: Server Registry, Models Reference It
- **Servers are registry entries.** The top-level `vllm-copilot.servers` setting is an explicit lookup table of server entries (`id`, `serverUrl`, optional `requestHeaders`, `serverType`, `displayName`). Each model entry in `vllm-copilot.models` has a required `id` and `server` (the registry entry's id) — models never carry URLs, auth headers, server types, or server labels.
- **The global settings are the `servers` registry plus the standalone toggles/diagnostics keys** (`enableFileLogging`, `logBodyLimit`, `systemMessageCapture`, `fixEmptyToolParameters`, `dashboard.pollIntervalMs`). Everything else lives inside a `servers` entry or a `models` entry.
- There is NO global `serverUrl`, `apiKey`, `requestHeaders`, or sampling params. The registry is not "a global server" — it is a table; nothing may resolve a server unless a model references its entry.
- The discovery logic must NOT probe a "global server" — it groups models by their `server` reference (resolved through the registry) and discovers from each server independently.
- There are NO deprecated legacy fields on `VllmConfig` — `serverUrl`, `apiKey`, and `requestHeaders` were removed outright by the registry migration.

### Version Compatibility
- **Only support newest versions.** Don't add workarounds for old versions unless explicitly requested. This goes for vLLM, VS Code, Copilot.
- Always use the latest version of any library or framework unless the user specifies otherwise. However, this version must be supported by Copilot, VS Code and vLLM.

### Build & Test
- **Compile:** `npm run compile` (runs `tsc -p ./`). This type-checks `src/` ONLY — it never proves the test files compile.
- **Test:** `npm test` (Vitest). Tests exist only as tripwires for real breakage (wire format, settings.json writes, provider lifecycle) — no coverage metric, no ceremony tests.
- **Package a VSIX:** `npm run build` = compile + vitest + **test typecheck** (`tsc -p test/tsconfig.json`) + vsce package. `test/tsconfig.json` extends the root tsconfig, so any compiler flag added at root also applies to `test/**`.
- **Gauntlet rule:** any tsconfig/compiler-flag change must be verified with `npm run build` (or at least `npx tsc -p test/tsconfig.json --noEmit`). Verifying only compile+test+dep:check once shipped an rc that failed its own build (35 dead test symbols, 2026-09-05). Neither `npm run rent` nor `npm run dep:check` type-checks `test/**`.

### Changelog Policy
- **Only issues a user actually experienced in a SHIPPED version get a `Fixed` entry.** A bug that was introduced and fixed within the same unreleased cycle is work-in-progress, not news — no entry, ever. This includes bugs seen only during the author's own rc/VSIX testing: an unpublished rc is not a shipped version.
- **New features deserve entries.** Internal refactors and implementation details do not.
- **Never compare against never-shipped intermediate behavior** ("before, during this rc, X happened"). If no user ever saw it, it never happened.
- **Intent before content, always.** A release gets a short intent paragraph directly under the version heading (the goal of the release, why it exists), before `### Added`. Each major structural change likewise states its goal first, then the change as its consequence. Never bury the why mid-paragraph, and state it once: the release-level intent paragraph replaces per-entry restatements of the same goal.
- Be terse in the changelog - this is for users to read. The commit messages can be verbose - those are for AI to read.
- **PowerShell: never put `$(...)` in a double-quoted git commit message** — PowerShell command-substitutes it and silently corrupts the message. Use single quotes.
- **Repo-specific changelog facts** (the epistemology behind them is upstream): **`package.json` `changelog` field points at the CHANGELOG.md blob URL, never at GitHub releases.** Marketplace versions and git releases are deliberately different things; not every version gets a git release. Note: a VSIX-installed extension shows the packaged `CHANGELOG.md` snapshot in the extension page's CHANGELOG tab and ignores the manifest field; the field only feeds Marketplace installs.

### code-review.md Policy
- `docs/code-review.md` tracks **live issues only**. When a finding is fixed, DELETE its entry in the same commit. No status sections, no "fixed by" annotations, no archives, no grades: git history holds what was done, and nobody reads accomplishment logs. Rejection lists, deferred architecture, and accepted product decisions stay (standing rulings, not history).

### Standing Review Rulings (owner-decided — do not re-propose)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fuzzifikation/vLLM-Copilot](https://github.com/fuzzifikation/vLLM-Copilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-04 -->

---
trigger: always_on
description: Build Otto as a secure, debuggable remote browser automation platform (controller -> relay -> extension node).
---

# AGENTS

## Mission
Build Otto as a secure, debuggable remote browser automation platform (controller -> relay -> extension node).

## Invariants
- Always require `targetNodeId` on commands. If there's only one connected node, CLI can auto-select it.
- Preserve terminal command outcomes: `completed`, `failed`, `timed_out`, or `cancelled`.
- Keep per-tab session execution serial and cross-tab execution parallel.
- Apply pre-ingress redaction for logs.
- Keep `chrome.debugger` features behind explicit opt-in flags.
- Keep command execution site-scoped and validate tab URL before running command logic.
- Keep `command.test` streaming command-native via returned stream listener manifests; avoid site-specific runtime listener managers.
- Keep runtime listener infrastructure generic (for example `network.http_intercept`) and keep site-specific stream parsing/fallback policy in command modules.
- Validate declared command input metadata before command execution; command handlers should receive sanitized input.
- For `otto test`, support optional command-level test hooks with execute fallback for simple commands.
- Never automate user credential submission; use explicit manual login handoff (`manual_login_required`).
- Keep listener lifecycle deterministic: `listener.subscribe` and `listener.unsubscribe` are terminal commands, while async `listener_update` events are correlated by the original subscribe `requestId`.
- Keep controller-to-node authorization node-owned: registered controller clients require explicit per-node ACL grants from the authenticated node before command routing.
- Keep `otto test` teardown signal-safe: Ctrl+C/SIGTERM should trigger `command_cancel` for active stream tests and attempt auto-opened `primitive.tab.close` before exit.
- Keep orphan cleanup owner-scoped: when controller sessions disconnect/time out, close only tabs owned by that controller identity.

## Canonical Commands
- Install deps: `npm install`
- Build all: `npm run build`
- Check types: `npm run check`
- Lint all: `npm run lint`
- Test all (where present, concise output): `NODE_OPTIONS='--test-reporter=dot' npm run -ws --if-present --silent test`
- Start relay daemon: `otto start`
- Start relay attached with logs (dev): `otto start --attached`
- Restart relay daemon: `otto restart`
- Stop relay daemon: `otto stop`
- Start MCP server: `otto mcp`
- Register with agent framework: `otto agent install <runtime>`
- Agent framework status: `otto agent status`
- Manual E2E harness: `npm run e2e:manual`
- Run relay dev: `npm run dev:relay`
- Run CLI dev: `npm run dev:cli`
- Run extension dev: `npm run dev:ext`
- Run docs dev: `npm run docs:start`
- Build docs site: `npm run docs:build`

## CI and Release Workflows
- Use Conventional Commits (`fix:`, `feat:`, `docs:`, etc.) so release-please can generate changelogs and version bumps correctly.
- CI package quality gates live in `.github/workflows/ci.yml` and should keep per-package checks independent.
- Release Please plus publish/asset automation lives in `.github/workflows/release-please.yml` and `release-please-config.json`.
- The release-please workflow includes a reconcile step that ensures merged release PRs always get their git tags and GitHub releases, even when release-please-action v4 skips tag creation. If a merged PR still has `autorelease: pending`, the reconcile step creates the tag and release and relabels the PR to `autorelease: tagged`.
- Publish and extension asset upload jobs use a `determine-release-tag` passthrough so they trigger from both release-please-created releases and reconciled (recovered) releases. The gate outputs an empty `release_tag` on ordinary pushes to main (no new release), which causes publish and upload jobs to skip. This prevents spurious publishes on non-release pushes.
- Keep semver-tagged extension assets attached to `v<version>` releases from the consolidated release workflow so `otto setup` download contract remains valid.
- Docs site deploy to GitHub Pages on docs path changes lives in `.github/workflows/docs-pages.yml`.

## Required Validation After Any Update
- Always run, in order, at the end of any code update:
- `npm run check`
- `npm run lint`
- `npm run build`
- `NODE_OPTIONS='--test-reporter=dot' npm run -ws --if-present --silent test`
- If a command fails, fix the issues and re-run until all pass.
- Write new tests for any new features or edge cases, and ensure they pass reliably.
- Update the documentation extensively to reflect any changes in behavior, architecture, or developer experience.
- For big changes, where it makes sense, also make concise updates to AGENTS.md.
- Never leave debug artifact files anywhere in the repo. This includes files like `test_results.txt`, `*.log`, scratch `*.json`, or any `*.html`/`*.md` outside `docs/` — at any directory level (repo root, `extension/`, `packages/`, etc.). Delete them immediately after use and do not commit them.

## Where To Change Things
- Protocol contracts: `packages/shared-protocol/src/index.ts`
- Relay routing and locks: `packages/relay/src/index.ts`
- CLI UX and command entrypoint: `packages/cli/src/index.ts`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [telepat-io/otto](https://github.com/telepat-io/otto) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->

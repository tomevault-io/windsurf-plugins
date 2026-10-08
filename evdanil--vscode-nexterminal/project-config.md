---
trigger: always_on
description: Always prefix shell commands with `rtk` in this repository. In command chains,
---

# AGENTS.md

## Shell commands

Always prefix shell commands with `rtk` in this repository. In command chains,
prefix each segment separately:

```bash
rtk git status && rtk npm run compile
```

Use raw commands only when debugging `rtk` itself or when explicitly requested.

## What this is

**Nexus Terminal** (`vscode-nexterminal`) — VS Code extension: SSH/telnet/serial
terminals, tunnels, SFTP, TFTP/DHCP. TypeScript strict (ES2022, CommonJS),
esbuild bundler, Vitest. Targets VS Code ^1.105 (Node 22 extension host,
`@types/node` 22); CI builds on Node 20 — either works locally.

## Commands

- `npm run compile` — type-check extension source only, with no emit.
- `npm run typecheck:test` — strict type-check of source and test files, with no emit. **No linter/formatter is configured**.
- `npm run build` — clean + type-check + all esbuild bundles + native artifact prep.
- `npm test` (coverage), `npm run test:unit`, `npm run test:integration`.
- Single test: `npx vitest run test/unit/nexusCore.test.ts`; by name: `npx vitest run -t "pattern"`.
- Integration tests run one at a time with 30s timeouts by design (real child processes/sockets) — never "fix" a timing flake with retries or skips; find the cause.
- `npm run package:vsix` fails on a fresh checkout (needs prebuilt PTY binaries for all six platforms from CI). You don't need a VSIX to develop.
- F5 (Extension Development Host) has no pre-launch build — run `npm run build` first or you get a stale `dist/`.

## Build constraint: five bundles

esbuild emits `dist/extension.js`, `dist/webExtension.js`, `dist/services/serial/serialSidecarWorker.js`, `dist/services/scripts/scriptWorker.js`, `dist/services/networkServers/networkServerDaemon.js`. A Node-only import reaching the browser graph breaks the web build. Worker/daemon bundles must not import `vscode` — the build enforces this: each out-of-host bundle carries a plugin (`scripts/buildConfigs.mjs`) that stops the build naming the importing file, held to the real configuration by `test/unit/esbuildWorkerIsolation.test.ts`.

## Never do these

- **Never put `[release]` on its own line in a commit message** — that exact line is the opt-in release trigger and publishes irreversibly to the Marketplace and Open VSX. Never write `[skip release]` (dead string, matches nothing). Two paths exist and they are not equal:
  - The **merge commit** marker is the maintainer's own route (`auto-release.yml`) — legitimate, but never added on anyone else's initiative.
  - A **branch commit** must never carry it: a squash body carries branch commit body lines verbatim, so a marker written on a branch becomes a marker on the merge and releases something nobody asked to release.
  - When you are told to cut a release, push the `v{version}` tag — `release.yml` and `publish-openvsx.yml` trigger on it directly, so the job is done without writing the marker at all. Verify first: version matches the CHANGELOG's top heading, the tag does not already exist, CI on `main` is green. The release is exactly that version, and the release notes are that one entry (`release.yml` publishes it as the GitHub Release body; it and `publish-openvsx.yml` both run `scripts/extractReleaseNotes.mjs` before publishing anything, and fail if the entry is missing, empty or only headings).
  - If your session cannot push tags (restricted agent sessions get HTTP 403 on a tag push while branch pushes work), and only when the maintainer explicitly asks for the release, use the merge-commit route instead: open a PR, and write the marker on its own line only in the squash-merge message you set at merge time, never in a branch commit. `auto-release.yml` then creates the tag and calls both publishing workflows and `recheck-open-prs.yml`. The same pre-tag checks apply.
  - After a release, confirm that each publish job (Marketplace and Open VSX) succeeded and that both listing pages show the new version — the workflow's top-level green is not enough. The next change PR then opens the next entry (see the version rule below).
  - After tagging, confirm `recheck-open-prs.yml` re-ran the `Version / changelog` gate on each open maintainer PR (it only warns on failure), and that a PR still on the released version is red until it opens the next entry. A PR merged between the tag and that re-run is not caught — check for bullets under the released heading.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [evdanil/vscode-NexTerminal](https://github.com/evdanil/vscode-NexTerminal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->

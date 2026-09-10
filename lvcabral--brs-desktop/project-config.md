---
trigger: always_on
description: handles `dev.zip`/`dev.bpk` upload and screenshots via `busboy`.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

`brs-desktop` is an Electron desktop wrapper around the **`brs-engine`** npm package (the BrightScript
simulation engine) plus **`brs-scenegraph`** (SceneGraph XML extension, alpha). Its job is to turn the
engine into a full Roku *device* simulator: network services (ECP/SSDP, web installer, telnet debugger),
device settings/persistence, menus, and an integrated Monaco-based code editor + console.

Language/runtime issues (BrightScript semantics, `roXXX` components) belong to `brs-engine`, not this repo.

## Commands

```bash
npm install            # postinstall runs electron-builder install-app-deps
npm run start          # dev mode: webpack watch (build/start.js) + spawns electron on first successful build
npm run build          # webpack dev build into app/
npm run release        # webpack production build into app/ (what CI runs)
npm run dist           # production build + electron-builder installers for the current platform -> dist/<version>/
npm run clean          # wipe app/
npm test               # vitest run — unit + service integration tests
npm run test:watch     # vitest watch mode
npm run test:coverage  # v8 coverage into coverage/
npm run lint           # eslint (flat config in eslint.config.mjs)
npm run lint:fix       # eslint --fix
npm run prettier       # prettier --check
npm run prettier:write # prettier --write
```

**Always run `npm run lint` and `npm run prettier` before committing**, and fix what they report —
CI runs both, and a formatting-only diff on a later PR re-attributes untouched code to that PR's new
code in SonarCloud (see below). `npm run lint:fix` and `npm run prettier:write` handle most of it.

Other `dist-*` scripts target Windows / Linux (appimage, deb, arm). Installers must be built on their
native OS.

CLI args can be appended to `npm run start` (e.g. `npm run start -- --devtools --console -m hd`); see
`docs/how-to-use.md` for the full list (`-o/-f/-m/-e/-r/-w/-p/-c/-d`).

## Tests

Tests run on **Vitest** (`test/**/*.spec.js`, config in `vitest.config.mjs`). Two layers:

- `test/unit/**` mirrors the `src/` tree and covers pure logic.
- `test/integration/**` boots the real ECP, web installer, telnet and debug servers in-process on
  **ephemeral ports** against a fake window, and drives them over real sockets.

There is **no E2E/Playwright layer**. Window behaviour, menus and anything visual still have to be
verified by running the app — `npm test` passing does not mean the UI works.

**Electron is never loaded.** `vitest.config.mjs` aliases `electron`, `@electron/remote`,
`@lvcabral/electron-preferences`, `@lvcabral/node-ssdp`, `network`, `electron-prompt` and
`electron-about-window` to stubs in `test/mocks/`. Mocking SSDP is what keeps UDP multicast out of CI.
`test/setup/global.js` polyfills `process.getSystemVersion()`, points `app.getPath("userData")` at a
temp dir, and installs a fresh `globalThis.sharedObject` before each test.

Two traps worth knowing:

- Several modules register `ipcMain` handlers **at module-evaluation time** and can never re-register.
  Do not call `ipcMain.removeAllListeners()` in a shared hook — it silently disables the code under
  test. Drive those handlers with `ipcMain.emit(channel, {}, payload)`.
- Routes that read bundled assets via `path.join(__dirname, …)` resolve to `src/` under vite-node
  rather than the webpack bundle's `app/`, so they fail in tests only. Those cases are marked.

When adding an IPC channel, a `gen*Xml` builder, or a debug command, add the matching test — the
whitelist-parity, XML and command-shell specs are the guardrails for those three contracts.

### Static analysis (SonarCloud)

Every PR is gated on SonarCloud's **new code** Quality Gate: A ratings for security, reliability and
maintainability, and hotspots 100% reviewed. The project key is `lvcabral_brs-emu-app`, which does not
match the repo name. Query findings with `resolved=false`, or already-closed issues come back too and
the list looks far worse than it is:

```bash
gh pr checks <PR>
curl -s "https://sonarcloud.io/api/issues/search?componentKeys=lvcabral_brs-emu-app&pullRequest=<PR>&resolved=false&ps=100"
```

**Moving code re-attributes it to new code**, so an extraction can pull an existing finding onto your
PR without you having written anything new. Check what a finding points at before assuming you caused it.

Rules this codebase trips most often, worth writing to up front:

| Rule | What it wants |
| --- | --- |
| S4790 | No weak hashes (MD5, SHA-1). Where a wire protocol mandates one, route every call through a single helper carrying the justification, so there is one documented exemption instead of many. |
| S5443 | No fixed path under a shared temp directory. Use `fs.mkdtempSync(path.join(os.tmpdir(), …))` — unique and owner-only. |
| S1313 | No hardcoded IP addresses. In fixtures and docs use the RFC 5737 ranges (`192.0.2.0/24`); loopback and subnet masks are fine. |
| S2699 | Every test needs at least one explicit `expect()`. A helper that throws on timeout does not count — assert the outcome after awaiting it. Empty `it.skip` bodies are flagged too; a comment explaining the gap says more. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lvcabral/brs-desktop](https://github.com/lvcabral/brs-desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-10 -->

---
trigger: always_on
description: Install, configure, verify, and troubleshoot JS Eyes browser automation for OpenClaw.
---


# JS Eyes

Use this skill to turn a ClawHub-installed `js-eyes` bundle into a working OpenClaw browser automation stack.

Treat `{baseDir}` as the installed skill root. The plugin path that must be registered in OpenClaw is `{baseDir}/openclaw-plugin`, not `{baseDir}` itself.

## Use This Skill When

- The user wants to install or configure JS Eyes from a ClawHub skill bundle.
- `js_eyes_*` tools are missing after installation.
- The browser extension is installed but still shows `Disconnected`.
- The user wants to verify the built-in server, plugin config, or extension connection.
- The user wants to discover or install JS Eyes extension skills after the base stack is working.
- The user wants to mount a **custom / external** extension skill (a directory outside the bundle that contains a `skill.contract.js`) into a running OpenClaw, or to verify that such a skill is actually loaded.
- The user wants to **author a new extension skill from scratch** — in that case, point them at the starter template and authoring guide (see `Authoring A New Extension Skill` below) rather than writing files from nothing.

## What Success Looks Like

A successful setup has all of the following:

1. `npm install` has been run in `{baseDir}` with Node.js 22 or newer.
2. OpenClaw loads `{baseDir}/openclaw-plugin` via `plugins.load.paths`.
3. `plugins.entries["js-eyes"].enabled` is `true`.
4. `tools.alsoAllow` (preferred) or `tools.allow` includes `js-eyes`, so the plugin's optional tools are actually exposed to the model.
5. The user can run `openclaw js-eyes status`.
6. The browser extension is connected to `http://<serverHost>:<serverPort>`, the popup **Server Token** field is populated (2.2.0+), and `js-eyes` with `action: browser/get-tabs` returns real tabs.
7. The bundled first-party extension skills under `{baseDir}/skills` are enabled by default in the JS Eyes host config, and the main plugin auto-loads enabled skills from `{baseDir}/skills` or the configured `skillsDir` (primary), plus any read-only directories listed in `extraSkillDirs` (extras). The user can later use `js-eyes` actions `skills/discover` and `skills/plan-install` to add more extension skills dynamically.
8. `js-eyes doctor` reports an acceptable security posture for this skill's deployment (token present, `allowAnonymous=false`, **`allowRawEval=true` in `~/.js-eyes/config/config.json`**, host bound to loopback, skill integrity OK). `doctor` will still label raw eval as insecure — that is expected when following this skill.

## Deployment Modes

Treat `{baseDir}` as the bundle or repository root that contains `openclaw-plugin/`, `skills/`, and the package manifests.

There are two supported complete deployment modes:

1. ClawHub / bundle deployment
   - `{baseDir}` is the installed JS Eyes bundle root.
   - Run `npm install` in `{baseDir}` so the plugin runtime can resolve its dependencies.
   - Register `{baseDir}/openclaw-plugin` in OpenClaw.

2. Source-repo / development deployment
   - `{baseDir}` is the root of a local `js-eyes` git clone.
   - Run `npm install` in the repo root, not inside `openclaw-plugin/`.
   - Point OpenClaw `plugins.load.paths` at the repo-root `openclaw-plugin` directory.
   - If you are debugging the browser side, load the extension from `extensions/chrome/` or `extensions/firefox/manifest.json` as appropriate.
   - **Restart scope**: only changes to files **under `openclaw-plugin/` itself** (or any `packages/*` module the plugin imports directly) require restarting / refreshing OpenClaw so the plugin module is re-`require`d. Edits to individual skill files (`skills/<id>/skill.contract.js`, an `extraSkillDirs` entry's contract, etc.) are picked up hot by the `SkillRegistry` + chokidar watcher — no restart needed.

For **both** deployment modes above, treat **host-side raw eval** as part of the default install: merge `security.allowRawEval: true` into `~/.js-eyes/config/config.json` (Windows: `%USERPROFILE%/.js-eyes/config/config.json`) **during** deployment — not in `openclaw.json`. The js-eyes server reads this file; the browser extension syncs the value on WebSocket handshake (2.5.1+). Restart the server / OpenClaw after changing it so `execute_script` raw JavaScript is not blocked with `RAW_EVAL_DISABLED`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [imjszhang/js-eyes](https://github.com/imjszhang/js-eyes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

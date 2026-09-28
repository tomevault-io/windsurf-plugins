---
trigger: always_on
description: This repository is a TypeScript CLI that runs Pi against local inference engines.
---

# AGENTS.md - localpi

This repository is a TypeScript CLI that runs Pi against local inference engines.

Before finishing code changes, run:

```bash
npm run check
```

Rules:

- Keep TypeScript strict. Do not use `any`; validate unknown JSON at the boundary.
- Keep local model discovery, Pi config generation, and process launching in separate modules.
- Add or update tests for behavior changes.
- Keep classifier-specific and final-schema workflows out of this repo. Those belong in callers such as localpager-agent.
- Follow `docs/design-principles.md`: simple, unopinionated, and customizable. Every default needs a flag or environment variable, and every interactive setting needs an in-session command when that is cheap to add.
- Do not commit generated output, local model responses, secrets, session files, or downloaded model files.
- Store persistent localpi user settings in `<state-dir>/settings.json`. Do not create a new top-level state file for each setting; add a field to the settings object instead. Separate files are only for distinct generated artifacts, runtime metadata, caches, logs, or external config formats.
- Follow the Slophammer agent entrypoint in `osolmaz/slophammer/docs/AGENT_ENTRYPOINT.md` when changing repo structure or quality gates.
- llama.cpp is the default local engine. Keep the `llama-cpp` provider first in `auto` discovery order so a loaded llama.cpp model wins automatic selection, and keep the managed `llama-server` fallback llama.cpp-based. Do not move another engine ahead of llama.cpp.
- Keep `@osolmaz/pi-factory` on the newest published version. When its API changes, adapt localpi instead of pinning an old release.
- Keep `@earendil-works/pi-coding-agent` as a devDependency on the newest Pi release, and keep the generated-extension typecheck test. Pi extension API drift must fail `npm test`, not a launch.
- Keep mutation testing out of the default gate. `npm run check` and the push/PR CI stay fast; `npm run mutate` is a manual, occasional run and the weekly `mutation` workflow.

## Pi Launch

- Keep the default Pi launch command at `npx -y @earendil-works/pi-coding-agent@latest`, so a normal launch runs the newest Pi release. Keep `--pi-command` and `LOCALPI_PI_CMD` as the escape hatches.
- Keep the Pi launch command a program plus arguments. pi-factory spawns it without a shell, so do not pass shell syntax, quoting, or environment prefixes through `--pi-command`.
- Split a typed Pi launch command with `parsePiCommand`, and keep quoted words together so a path with spaces survives.
- Keep the default skills mode at `own`. Localpi launches Pi with `--no-skills` and loads only `<state-dir>/pi-skills/`, so shared skill directories such as `~/.agents/skills` stay out of a local session. `--skills ambient` restores Pi's own discovery, and `--skills off` loads nothing.

## llama.cpp

llama.cpp is localpi's default and preferred local engine.

- The `llama-cpp` provider probes `http://127.0.0.1:8080/v1` by default and is listed first in `auto` discovery.
- It reads llama.cpp `/v1/models` entries. A model with `status.value` `loaded`, or with no status at all, is usable. A model with `status.value` `unloaded` is offered as startable only when `/props` reports `models_autoload`.
- External llama.cpp servers are never started, stopped, or unloaded by localpi. Only the localpi-owned managed `llama-server` process is managed.
- Keep llama.cpp-specific parsing in the shared model-discovery layer instead of duplicating it in callers.
- Read image input from the server, never from the model name. A llama.cpp entry that lists `image` in `architecture.input_modalities` makes the Pi model config say `input: ["text", "image"]`; every other model stays `input: ["text"]`. A model profile may state `capabilities.image` for a server that reports nothing, and the profile wins in both directions. A model that takes images needs a multimodal projector on the server (`--mmproj`), and localpi does not start or configure that server.

## Status Display

The status display follows `docs/design-principles.md`: simple, unopinionated, customizable.

- Keep the status line to one row. Localpi replaces Pi's footer through `ctx.ui.setFooter` and renders one line: working directory and branch, token totals and cache, context use, then the engine next to the model. Do not call `setStatus`, which would add another row.
- Show a number in one place at a time. The status line drops the context group and the last rate while the model runs, because the live line shows both then.
- Keep the live rate in the working line and the rate of the last finished turn in the status line while the model is idle, so the speed stays visible after an answer ends. Publish that turn between the two generated extensions through the typed `localpiStats` global. Do not use a second status item for it, because a status item adds a row. Drop the rate first when the line is too narrow.
- Build the engine label from the launch-time provider map (`engineEntries`) and follow the current model's provider. Never guess an engine from a model name or a base URL. When the engine is unknown, show no label.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [osolmaz/localpi](https://github.com/osolmaz/localpi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->

---
trigger: always_on
description: <!-- FOR AI AGENTS - Human readability is a side effect, not a goal -->
---

<!-- FOR AI AGENTS - Human readability is a side effect, not a goal -->
<!-- Managed by agent: keep sections and order; edit content, not structure -->
<!-- Last updated: 2026-09-25 | Last verified: 2026-09-25 -->

# AGENTS.md

**Precedence:** the **closest `AGENTS.md`** to the files you're changing wins. This is the only one in the repo, and
the only agent rules file.

## Project

jev-router is a local pass-through model router for Claude Code (Anthropic Messages) and the Codex CLI (OpenAI
Responses). On each new human message it asks Jev, TypeSafe AI's System One decision model, which tier the work
needs, and routes the session to a fast, balanced or frontier model. Dependency-free Node ESM (`.mjs`, Node >= 22).
No build step and no bundler: TypeScript only type-checks the JSDoc. The primary setup is a plain Mac or Linux
machine, set up with `jev-router setup` (a service plus Claude Code's settings); Docker Sandboxes are an optional
variant.

| Fact | Value |
| --- | --- |
| Entry point | `bin/jev-router.mjs` calls `main(argv)` in `src/cli.mjs`, whose `HELP` is the usage `jev-router help` prints |
| Server | `createRouter`, `describeConfig`, `report` and `VERSION` in `src/router.mjs` |
| Live view | `serve --ui [<host>:]<port>` (or `JEV_ROUTER_UI`) feeds `createUiServer` (`src/ui.mjs`) in-process through `publish`; `jev-router ui [log]` feeds it with `LogTail` from a log file. It serves `ui/` (`index.html`, `app.css`, `app.js`, `favicon.svg`) plus server-sent events, on `127.0.0.1:4100` by default; `--ui-token` or `JEV_ROUTER_UI_TOKEN` makes it ask for a token. `ui/tsconfig.json` type-checks the browser code |
| Modules | `src/config.mjs` defaults and validation; `src/jev.mjs` state, questions, channels, policy; `src/messages.mjs` human turns, wrapper tags, tier tags; `src/secrets.mjs` scanner and redaction; `src/sessions.mjs` persistent store; `src/usage.mjs` usage tap and prices; `src/logfile.mjs` log appends and rotation; `src/ui.mjs` live view server; `src/files.mjs` where files live (XDG paths, config lookup), programs on PATH, shell quoting; `src/net.mjs` addresses, ports and the `/healthz` probe; `src/envfile.mjs` loading the env file, and writing keys into it in place; `src/claude.mjs` Claude Code's variables and its settings file; `src/prompt.mjs` setup's questions (a line reader for pipes, raw-mode secrets on a terminal); `src/service.mjs` the launchd agent and the systemd user unit, rendered from `examples/service/`; `src/install.mjs` npx detection, the global install and the package name; `src/setup.mjs` the `setup` flow, the manifest, the root check (`rootProblem`) and the gateway check; `src/uninstall.mjs` the `uninstall` flow; `src/types.d.ts` shared JSDoc types, imported as `/** @import { Config } from './types.js' */` |
| Configs | `config/default.json`, `config/anthropic-only.json`, `config/anthropic-fable.json` (the Anthropic-only config plus a `max` tier on Fable 5.1; a unit test holds it to exactly that difference). `PACKAGED_CONFIGS` in `src/files.mjs` names them `ollama`, `claude` and `fable` for `setup --models` and `init --models`; `packagedModels` tells an unchanged copy of any released version by its digest in `PACKAGED_DIGESTS`, which setup may replace, and `inPackage` keeps setup and `init` from writing into the package. Lookup: `--config`, `JEV_ROUTER_CONFIG`, `$XDG_CONFIG_HOME/jev-router/config.json` (`~/.config/jev-router/config.json`, written by `init`), then `config/default.json`. Every key: `docs/configuration.md` |
| Keys | The environment, or an env file of `KEY=VALUE` lines: `--env-file`, else `JEV_ROUTER_ENV_FILE`, else `$XDG_CONFIG_HOME/jev-router/env` when it exists (source "default location"; `setup` writes it, mode 0600, with a `JEV_ROUTER_HOST` or `JEV_ROUTER_PORT` from the shell); naming `/dev/null` turns it off. loaded with `process.loadEnvFile` before anything reads the environment (`loadEnvFile` in `src/envfile.mjs`). Variables already set win; `launch` keeps the file's variables away from the agent |
| Setup | `jev-router setup` (`runSetup` in `src/setup.mjs`): asks everything first (`Prompter` in `src/prompt.mjs`), checks the Jev key with one `JevClient.decide`, then writes the config, the env file (`saveEnvValues`), the service (`src/service.mjs`, after a global `npm install -g` when run from npx: `src/install.mjs`) and, once the router answers, Claude Code's `settings.json` with a backup. `$XDG_CONFIG_HOME/jev-router/setup.json` records its changes without secrets, written before `settings.json`; `jev-router uninstall` (`runUninstall` in `src/uninstall.mjs`) reads it. It never points Claude Code at the router while Claude Code goes to another gateway with credentials for it (`claudeCredentials`, `foreignBaseUrl` in `src/claude.mjs`), and `launch claude` and `env claude` refuse then too. Both commands refuse root on another user's behalf |
| Log | JSON lines on stdout, and to `--log-file`, else `JEV_ROUTER_LOG_FILE`, else the config's `logFile`, through `appendLogLine` (`src/logfile.mjs`): a file rotates to `<file>.1` at `logMaxBytes` (50 MiB; 0 turns rotation off) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dirien/jev-router](https://github.com/dirien/jev-router) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->

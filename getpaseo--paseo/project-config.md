---
trigger: always_on
description: Paseo is a mobile app for monitoring and controlling your local AI coding agents from anywhere. Your dev environment, in your pocket. Connects directly to your actual development environment — your code stays on your machine.
---

# CLAUDE.md

Paseo is a mobile app for monitoring and controlling your local AI coding agents from anywhere. Your dev environment, in your pocket. Connects directly to your actual development environment — your code stays on your machine.

**Supported agents:** Claude Code, Codex, and OpenCode.

## Repository map

This is an npm workspace monorepo:

- `packages/server` — Daemon: agent lifecycle, WebSocket API, MCP server
- `packages/app` — Mobile + web client (Expo)
- `packages/cli` — Docker-style CLI (`paseo run/ls/logs/wait`)
- `packages/relay` — E2E encrypted relay for remote access
- `packages/desktop` — Electron desktop wrapper
- `packages/website` — Marketing site (paseo.sh)

## Documentation

| Doc                                                  | What's in it                                                                      |
| ---------------------------------------------------- | --------------------------------------------------------------------------------- |
| [docs/architecture.md](docs/architecture.md)         | System design, package layering, WebSocket protocol, agent lifecycle, data flow   |
| [docs/coding-standards.md](docs/coding-standards.md) | Type hygiene, error handling, state design, React patterns, file organization     |
| [docs/testing.md](docs/testing.md)                   | TDD workflow, determinism, real dependencies over mocks, test organization        |
| [docs/development.md](docs/development.md)           | Dev server, build sync gotchas, CLI reference, agent state, Playwright MCP        |
| [docs/release.md](docs/release.md)                   | Release playbook, draft releases, completion checklist                            |
| [docs/custom-providers.md](docs/custom-providers.md) | Custom provider config: Z.AI, Alibaba/Qwen, ACP agents, profiles, custom binaries |
| [docs/android.md](docs/android.md)                   | App variants, local/cloud builds, EAS workflows                                   |
| [docs/design.md](docs/design.md)                     | How to design features before implementation                                      |
| [SECURITY.md](SECURITY.md)                           | Relay threat model, E2E encryption, DNS rebinding, agent auth                     |

## Quick start

```bash
npm run dev                          # Start daemon + Expo in Tmux
npm run cli -- ls -a -g              # List all agents
npm run cli -- daemon status         # Check daemon status
npm run typecheck                    # Always run after changes
npm run lint                         # Always run after changes
npm run format                       # Auto-format with Biome
npm run format:check                 # Check formatting without writing
```

See [docs/development.md](docs/development.md) for full setup, build sync requirements, and debugging.

## Critical rules

- **NEVER restart the main Paseo daemon on port 6767 without permission** — it manages all running agents. If you're an agent, restarting it kills your own process.
- **NEVER assume a timeout means the service needs restarting** — timeouts can be transient.
- **NEVER add auth checks to tests** — agent providers handle their own auth.
- **NEVER run the full test suite locally.** The test suites are heavy and will freeze the machine, especially if multiple agents run them in parallel. Rules:
  - Run only the specific test file you changed: `npx vitest run <file> --bail=1`
  - Never run `npm run test` for an entire workspace unless explicitly asked.
  - If you must run a broad suite, pipe output to a file and read it afterward: `npx vitest run <file> --bail=1 > /tmp/test-output.txt 2>&1` then read the file.
  - Never re-run a test suite that another agent already ran and reported green — trust the result.
  - For full suite verification, push to CI and check GitHub Actions instead.
- **Always run typecheck and lint after every change.**
- **Build workspace packages before diagnosing cross-package type errors.** This repo consumes generated declarations across workspaces. If typecheck fails in a package that depends on another workspace (especially CLI depending on server/daemon types), rebuild the owning package first so `dist` declarations are current:
  - `npm run build:daemon` — rebuild highlight, relay, server, and CLI when daemon/server/CLI types may be stale.
  - Do not patch inferred callback parameters or add local duplicate types just to silence stale declaration errors.
- **Run `npm run format` before committing.** This repo uses Biome for formatting. Do not manually fix formatting — let the formatter handle it.
- **Always use npm scripts for linting and formatting.** Do not run tools directly with `npx eslint`, `npx oxfmt`, `npx oxlint`, or package-local binaries. For targeted checks, pass file paths through the npm script:
  - `npm run lint -- packages/app/src/components/message.tsx`
  - `npm run format:files -- CLAUDE.md packages/app/src/components/message.tsx`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [getpaseo/paseo](https://github.com/getpaseo/paseo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->

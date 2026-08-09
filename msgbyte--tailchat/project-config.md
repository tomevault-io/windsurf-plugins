---
trigger: always_on
description: This file applies to the entire repository. It is guidance for coding agents,
---

# Tailchat Agent Guide

## Scope and priorities

This file applies to the entire repository. It is guidance for coding agents,
not a replacement for `README.md`, `README.zh.md`, or the long-form website
documentation. A direct user request always takes precedence.

Tailchat is a noIM platform: messaging is the collaboration core and plugins
are first-class product extensions. Preserve that model when choosing where a
change belongs.
The primary runtime flow is:

1. The web entry configures shared adapters and loads MiniStar plugins.
2. React UI calls APIs, Socket.IO, state, and domain helpers in `client/shared`.
3. The gateway maps HTTP or socket requests to Moleculer `TcService` actions.
4. Services validate, authenticate, authorize, coordinate, and access models.
5. Notifications return through Socket.IO and update shared client state.

Do not bypass this flow by coupling core UI to plugin internals, accessing data
around service actions, or relying on client-side authorization.

## Repository map

- `client/web`: browser application, routes, browser integration, and plugin host.
- `client/shared`: cross-platform API, socket, state, hooks, models, and utilities.
- `client/packages`: frontend design system, client SDK, and build-time tooling.
- `client/web/plugins`: pure frontend plugins and their manifests.
- `server/services`: core and OpenAPI Moleculer services.
- `server/models`: Typegoose persistence models.
- `server/packages/sdk`: `TcService`, gateway, permission, runner, and SDK contracts.
- `server/plugins`: backend or full-stack plugins, sometimes with embedded web UI.
- `server/admin`: separate Vite/React frontend and Express admin service.
- `packages/types`: published structures shared across package boundaries.
- `client/desktop`: current Electron shell; it loads a Tailchat web deployment.
- `client/mobile`: React Native WebView shell and native bridges.
- `client/desktop-old`: legacy implementation; do not use it as the default target.
- `apps`: non-core CLI, GitHub app, OAuth demo, and embeddable widget.
- `website`: Docusaurus documentation and marketing site.

Read the nearest package scripts and local documentation before editing. Some
subprojects have distinct build systems even when their source lives here.

## Working protocol

1. Start with `git status --short --branch --untracked-files=all`.
2. Trace the real entry point, caller, contract, and tests before changing code.
3. Follow an existing service, model, component, or plugin pattern when possible.
4. Make the smallest coherent change; avoid unrelated cleanup and refactoring.
5. Preserve unrelated staged, unstaged, ignored, and untracked user work.
6. Add or update focused tests beside the behavior being changed.
7. Run checks proportional to the affected subsystem.
8. Report exact commands, failures, skipped checks, and environment blockers.
9. Never expose passwords, JWT secrets, API tokens, admin credentials, or `.env` contents in source, logs, tests, generated artifacts, or responses.

Do not run repository-wide `pnpm lint:fix` by default. It can rewrite files
outside the task. Lint or format only files in scope.

Do not edit `node_modules` or generated `dist`/build/coverage/runtime output as source. Tracked tooling under `client/build` and `client/web/build` is source.

## Toolchains and package ownership

CI and Docker define the main baseline: Node.js 18 and pnpm 8.15.8. A newer
machine-local Node or pnpm version does not expand supported compatibility.

Use pnpm for the root workspace. Install with:

```bash
pnpm install --frozen-lockfile
```

The root `pnpm dev` starts the server and web development processes. Local
development also needs the configured MongoDB, Redis, and MinIO dependencies;
the default web and gateway ports are 11011 and 11000.

`client/desktop`, `client/mobile`, and `client/desktop-old` keep independent
Yarn lockfiles. Run Yarn from the owning directory and do not rewrite those
locks with pnpm.

Dependency changes must update the owning manifest and matching lockfile. Avoid
lockfile churn unrelated to the requested dependency change.

## Frontend rules

- Keep plugin initialization before application rendering in `client/web`.
- Use `client/shared` only for behavior that is genuinely cross-platform.
- Keep DOM, browser, Electron, and React Native behavior in platform layers.
- Reuse shared request and socket wrappers; preserve token and error handling.
- Keep remote-event to local-state synchronization in the established Redux flow.
- Reuse existing design components and registration APIs before adding globals.
- User-visible core strings use existing `t`, `Trans`, and translation workflows.
- Frontend permission checks control presentation only, never final authorization.

The web TypeScript configuration excludes plugin source. A successful core web
type check does not prove that frontend plugins compile; run the MiniStar build
when plugin code or plugin-facing exports change.

## Backend rules

- Implement service behavior through `TcService` and a stable `serviceName`.
- Register actions in `onInit` and declare concrete parameter validation.
- Keep actions authenticated unless a public endpoint is deliberately reviewed.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [msgbyte/tailchat](https://github.com/msgbyte/tailchat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->

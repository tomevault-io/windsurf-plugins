---
trigger: always_on
description: GSV is a distributed AI operating environment built from five main pieces:
---

# GSV Agent Guidelines

GSV is a distributed AI operating environment built from five main pieces:

- a gateway worker that owns the kernel, process runtime, auth, package management, adapters, and inference routing
- a web shell that hosts the desktop and embedded apps
- standalone adapter workers for external platforms such as WhatsApp and Discord
- a Rust CLI for users, devices, deployment, and administration
- the `ripgit` worker for git-backed storage and content operations

This document is the working orientation for the current repository. It should help an agent understand where things live, how they deploy, and how to validate the right surface after a change.

## Repository layout

```text
gsv-app-runtime/
├── gateway/
│   ├── src/
│   │   ├── kernel/          # syscall dispatch, auth, config, adapters, packages
│   │   ├── process/         # Process DO runtime, store, queue, checkpoint, media
│   │   ├── syscalls/        # gateway syscall surface and process-local types
│   │   ├── inference/       # provider/model integration
│   │   ├── fs/              # filesystem and ripgit integration
│   │   ├── downloads/       # self-hosted CLI download/install support
│   │   ├── auth/            # passwords, tokens, setup auth
│   │   ├── shared/          # worker/DO bridge utilities
│   │   └── protocol/        # WS and RPC frame types
│   ├── wrangler.jsonc
│   └── package.json
├── builtin-packages/        # builtin apps synced from system/gsv
├── shared/                  # shared SDKs, contracts, and app-link types
├── web/
│   ├── src/                # desktop shell, host bridge, setup/login UI
│   ├── public/
│   └── package.json
├── adapters/
│   ├── whatsapp/
│   ├── discord/
│   └── test/
├── cli/
├── ripgit/
├── scripts/
├── templates/
├── docs/
├── README.md
└── CHANNELS.md
```

## Runtime model

### Gateway

The gateway is the control plane.

It is responsible for:
- websocket connections
- identity and auth
- syscall dispatch
- process lifecycle
- package sync and package permissions
- adapter routing
- system configuration
- model/provider dispatch

The most important directories are:
- `gateway/src/kernel/*`
- `gateway/src/process/*`
- `gateway/src/syscalls/*`
- `gateway/src/inference/*`

### Processes

A process is the unit of agent execution.

The process runtime owns:
- user/assistant/tool message history
- queued incoming messages
- pending tool calls
- checkpointing
- process-scoped media storage and hydration

Key files:
- `gateway/src/process/do.ts`
- `gateway/src/process/store.ts`
- `gateway/src/process/checkpoint.ts`
- `gateway/src/process/media.ts`

Important syscalls:
- `proc.spawn`
- `proc.send`
- `proc.history`
- `proc.abort`
- `proc.reset`
- `proc.kill`

### Web shell

The web shell is the desktop UI.

It owns:
- login and setup flows
- the desktop frame
- the iframe host bridge for builtin apps
- app/window orchestration

Key files:
- `web/src/session-ui.ts`
- `web/src/session-service.ts`
- `web/src/host-bridge.ts`
- `web/src/gateway-client.ts`

### Builtin apps

Builtin apps live under `builtin-packages/*`.

Examples:
- `chat`
- `files`
- `shell`
- `devices`
- `processes`
- `control`
- `packages`
- `adapters`

They are synced from `system/gsv` into the running system. A builtin app change is not applied by redeploying the gateway worker alone.

### Adapter workers

Adapter workers are separate deployables.

Each one owns its platform-specific behavior:
- auth and account state
- inbound event normalization
- outbound message delivery
- adapter-specific identity normalization

Gateway calls adapters through service bindings. Adapter workers call back into gateway through gateway RPC entrypoints.

### ripgit

`ripgit` provides git-backed storage and content operations used by the gateway filesystem and package/repo flows.

## Update model

This is the most important operational distinction in the repo.

### `gateway/src/*`

You changed gateway worker code.

Use:
```bash
cd gateway
npm run deploy
```

Local dev:
```bash
cd gateway
npm run dev
```

### `web/src/*` or `web/public/*`

You changed the web shell.

Use:
```bash
cd web
npm run build
```

Then redeploy however the built web bundle is served in that environment.

Local dev:
```bash
cd web
npm run dev
```

### `builtin-packages/*`

You changed a builtin app.

Use:
```bash
git push <remote> HEAD:main
cargo run -- -u root packages sync
```

If the package is a new builtin, the running gateway code must already know about that builtin package.

### `adapters/*`

You changed an adapter worker.

Deploy that specific worker:

```bash
cd adapters/whatsapp
npm run deploy
```

```bash
cd adapters/discord
npm run deploy
```

### Combined changes

If a change spans multiple layers, update each one explicitly.

Examples:
- `gateway/src/*` + `builtin-packages/*`
  - redeploy gateway
  - sync builtins
- `gateway/src/*` + `web/src/*`
  - redeploy gateway
  - rebuild/redeploy web shell
- `builtin-packages/*` + `adapters/*`
  - sync builtins
  - redeploy that adapter

## Development commands

### Dependency bootstrap

```bash
./scripts/setup-deps.sh
```

### Local multi-worker stack

```bash
./scripts/dev-stack.sh
```

### Gateway

```bash
cd gateway
npm run dev
npx tsc --noEmit
npm run test:run

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [deathbyknowledge/gsv](https://github.com/deathbyknowledge/gsv) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->

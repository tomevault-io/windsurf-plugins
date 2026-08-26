---
trigger: always_on
description: This repository owns the desktop product around an unmodified DeepSeek Harness checkout.
---

# DSH Desktop repository rules

This repository owns the desktop product around an unmodified DeepSeek Harness checkout.

## Prerequisites and setup

- Use Node.js `^22.19.0` or `>=24.0.0` and the root Yarn `4.18.0` release through Corepack.
- Initialize the pinned upstream checkout with `git submodule update --init --recursive`.
- Install root dependencies with `corepack yarn install --immutable`.

## Build, run, and verify

- Start the isolated local Desktop (own `~/.dsh-acryl` home, advanced mode, Development Canvas) with `corepack yarn dev` or `corepack yarn local`.
- Use `corepack yarn dev:shared` only when you intentionally want the installed app's `~/.dsh` home.
- Fast headless loop: `corepack yarn typecheck`, `corepack yarn test`, or both via `corepack yarn verify`.
- Typecheck, test, then isolated GUI: `corepack yarn lifecycle`.
- Build the desktop package with `corepack yarn build`.
- Run the complete headless gate with `corepack yarn check`.
- Run upstream operations through the root scripts, such as `corepack yarn upstream:build`.

- `deepseek-harness/` is a pinned upstream Git submodule. Never edit files inside it from a desktop feature branch.
- `dsh-plugin-desktop/` owns the Cordis Host and Client faces, Electron bootstrap, packaging, and release tests.
- `dsh-community-fabric/` owns the community interoperability RFC. Until schemas and a reviewed reference adapter exist, it remains a private documentation scaffold and must not declare loadable DSH or package entry points.
- `dsh-community-market/` is an implemented private Host/Client package. It is an optional Desktop Market provider, disabled by default, and must continue to use ordinary DSH/Cordis, profile, and Desktop service contracts rather than a parallel plugin runtime.
- The outer repository and all owned packages use the root Yarn release with `nodeLinker: node-modules`.
- The upstream submodule keeps its own pnpm workspace. Run upstream commands through the root `upstream:*` scripts, whose Yarn portable-shell commands enter the submodule before invoking Corepack.
- Compatibility mode must run the upstream default client without overrides. Advanced presentation belongs to desktop-owned client plugins and may replace documented slots or services through profile composition.
- Keep graphical application launch explicit. Builds, typechecks, unit tests, and Loader smokes must remain headless-safe.
- During the current small-team rapid-development phase, work directly on `main` unless the user explicitly requests a branch or pull request. Do not create routine PRs.
- Commit every important coherent change promptly so regressions can be reverted to a precise checkpoint. Prefer several focused commits over rare large snapshots.
- Record every important product, architecture, workflow, or operational evolution in [`docs/DEVELOPMENT-LOG.md`](docs/DEVELOPMENT-LOG.md). Commit the implementation first, then add its full canonical commit hash and human-readable explanation to the log in a separate documentation checkpoint. Update logged hashes after any history rewrite or squash.
- Commit before major changes of direction and keep the submodule pin update separate from desktop behavior changes.
- Keep the repository topology and package-manager split consistent with the [owning Agent Note](.agents/notes/implemented/process/2026-08-15-pinned-upstream-and-isolated-yarn-workspace.md).

## Cordis development protocol

Before changing a plugin, service, tool, provider, event hook, Host route,
Client contribution, or Loader composition, read
[`docs/cordis/cordis_system_guide_for_coding_agents.md`](docs/cordis/cordis_system_guide_for_coding_agents.md).
Then inspect the owning pinned Harness subsystem documentation and package
types/source. The local vendored Harness behavior is authoritative over generic
Cordis examples.

Before implementation, write the Cordis mini-design in the active plan or
research note:

1. **Capability and plugin boundary** — what domain owns it and why it needs
   independent lifecycle/configuration/replacement.
2. **Provides and consumes** — services, tools, events, durable facts, hard
   `inject` requirements, and intentionally optional `ctx.get()` dependencies.
3. **Effects and disposal** — every activation-owned resource, its disposer,
   required cleanup order, cancellation, and quiescence behavior.
4. **Configuration and composition** — validated runtime schema, stable Loader
   row ids, scopes/isolation, and provider replacement behavior.
5. **Events and durability** — dispatch mode, explicit waterfall `next()`
   semantics, and which replay-critical facts belong in durable session state.
6. **Verification** — real Loader activation plus PENDING/reactivation,
   provider replacement, disposal, repeated mount/reload, and leak checks.

Use function plugins by default and `Service` classes for direct named
capabilities. Consumers depend on stable service interfaces through `inject`,
not concrete providers or YAML row order. Cordis/Harness registrations are
lifecycle-owned; raw timers, watchers, sockets, PTYs, subprocesses, routes, and
subscriptions are acquired inside one owning `ctx.effect()` and fully released
by its disposer.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [acryldev/acryl](https://github.com/acryldev/acryl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->

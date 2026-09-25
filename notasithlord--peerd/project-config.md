---
trigger: always_on
description: > Persistent project context for Codex. Read this first on every
---

# AGENTS.md — peerd

> Persistent project context for Codex. Read this first on every
> session. Equivalent to OpenCode's `AGENTS.md` pattern.

---

## What peerd is

A browser-native AI agent harness, shipped as a Chrome/Firefox extension.
The agent runs entirely in the user's browser. It talks to a model API
directly (BYOK — bring your own key), drives the browser's tabs and
DOM, and can run shell commands in a sandboxed Linux VM compiled to
WebAssembly. No backend, no telemetry, no account.

peerd is **0.x — experimental beta** (breaking changes likely; no "V1"
commitment — versions stay 0.x until the surface stabilizes). The
initial feature buildout is complete and integrated; current work is
store-readiness polish and field hardening. Stack:
Manifest V3, Chrome and Firefox (via `webextension-polyfill`), vanilla
JS, **no
build step**.

---

## The codebase is five modules

Each maps to one letter and color in the brand wordmark:

| Letter | Color | Module | Role |
|---|---|---|---|
| `p` | cyan    | `peerd-provider/`     | Model adapters (Anthropic, OpenRouter, OpenAI, Z.ai GLM keyed; Ollama and the on-device WebGPU runner keyless - `registry.js` is the live list) |
| `e` | red     | `peerd-egress/`       | Security: vault, allowlist (`safeFetch`), denylist, audit |
| `e` | amber   | `peerd-engine/`       | Execution instances - Sandboxes. FOUR kinds run in their own visible tab: WebVMs (CheerpX Linux), Notebooks (sealed JS worker + OPFS), Pods, Apps (opaque-origin iframe). A fifth, the **headless worker** (`script`), runs the Notebook's sealed worker offscreen with no tab - the agent's own quick compute, and the only kind with no registry because it persists no instances. Also here: browser-native Git (`repository/`, exported from `index.js`). The sandbox is the isolate; a tab is one way to host it (taxonomy in `registry-factory.js`). |
| `r` | green   | `peerd-runtime/`      | Agent loop, tools + per-environment actors (`message_actor`), sessions, profiles, skills, memory, permissions (Plan/Act), goal mode (autonomous loop), composer, cost, transfer, voice, clock, web tool policy |
| `d` | magenta | `peerd-distributed/` | The dweb. An always-on P2P base network (offscreen mesh + DHT + gossip), did:key identity, signed content addressing, the dwapp bridge, and a peer-to-peer app store that **users AND the agent** build, share, and run dwapps on. Chrome preview only until Firefox has a mesh host. |

The extension *chassis* lives outside these modules: `background/`,
`offscreen/`, `sidepanel/`, `home/`, `options/`, `engine-tabs/`,
`permissions/`, `eval/`, `shared/`, `tests/`, `vendor/`, `icons/`. Each
tab-hosted `peerd-engine` execution kind owns a dedicated page under
`engine-tabs/` (`vm-tab/`, `notebook-tab/`, `pod-tab/`, `app-tab/`) - grouped so the
four engine host surfaces sit together; `permissions/` hosts user-gesture surfaces such as
the mic-permission grant page; `eval/` is the live end-to-end eval
harness. (There is no `content/` directory - DOM work happens via
injected functions, not a persistent content script.) Outside
`extension/`, the repo also carries `signaling-node/` (the dweb
rendezvous server shells sharing the pure signaling reducer). The
peerd.ai site lives in its own repo now (`NotASithLord/peerd-site`); it
vendors snapshots of the VM-demo runtime + the `peerd-distributed`
transport. Release packaging generates auto-update feeds as release assets;
the site repository downloads and deploys them.

The brand IS the architecture: the five letters are the five modules.
A new top-level peerd-* directory needs a reason the five letters cannot
absorb. Stop and reconsider before adding one. The cross-module import
gate matches on `peerd-[a-z-]+/`, hyphen included; a name that falls
outside that pattern is a name outside the boundary.

---

## What to read, in order

The code is the spec. There is no separate design-doc corpus; the
prose orientation is this file, and the rest is the source itself.

1. **`AGENTS.md`** (this file) — orientation. Start here every session.
2. **The module code** — `extension/peerd-provider/`, `peerd-egress/`,
   `peerd-engine/`, `peerd-runtime/`, `peerd-distributed/`. Each
   module's `index.js` is its universal public surface; read it first, then the
   files it exports. An explicit environment entry point such as
   `background.js` may expose a cold-start-safe subset for that host.
3. **The code.** For concrete behavior (vault crypto, denylist matcher,
   agent loop, tool dispatcher, prompt-injection defenses, the manifest,
   the MV3 keepalive trick), read the source in the relevant module. It
   is authoritative; prose drifts, code does not.

---

## Non-negotiable conventions

- **Vanilla JS, ES modules, no development build step.** `extension/` runs
  directly in the browser. The dev loop is *load unpacked -> refresh*; there
  is no source bundler, transpiler, or generated runtime tree. Store and
  preview packaging may compact authored static cold-path modules only in its
  disposable staging copy. It must preserve module boundaries and names, lazy
  imports, and vendor bytes. `packaging/preflight.ts` is the release authority.
- **Generated files — don't hand-edit.** `extension/manifest.json` and

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NotASithLord/peerd](https://github.com/NotASithLord/peerd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->

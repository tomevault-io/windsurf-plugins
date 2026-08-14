---
trigger: always_on
description: **ALL TypeScript work** use this Deno + Hono + JSR "ABC-layering" style. Always
---

# publicdomainrelay typescript

**ALL TypeScript work** use this Deno + Hono + JSR "ABC-layering" style. Always
active. Pattern applies per repo. Poly repo: all org repos live under dir holding
this file = **org root**.

Always run `./scripts/find-all-package.ts | yq -P` on session start to see where
everything lives.

## Open Architecture — docs-as-code blueprint

`open-architecture/` is a **docs-as-code** translation of
`open_architecture_today.md` into TypeScript stub functions organized in ABC
layers. Each function = one paragraph of the document. Function body = calls to
sub-concepts that paragraph depends on. **The call graph IS the architecture.**
Walk the references and you walk her whole reasoning. Explore with:

```
codegraph explore "whatAliceIs theInfiniteLoop puttingItTogether"
```

**STATUS_REPORT.md** (`open-architecture/STATUS_REPORT.md`) maps every
docs-as-code stub to its production implementation across the polyrepo.
Read it to understand what's built vs what's still stub.

**Regenerate the status report**: ask Claude to "regenerate the status report"
or invoke the `status-report` agent. Fans out 5 cavecrew-investigator subagents
across the polyrepo, maps findings to stubs, writes `STATUS_REPORT.md`.

## RFP flow is the spine — NEVER bypass it

This entire project exists to provision compute through the **RFP / market
flow**. A requester posts a `compute.vm` record + a signed `market.rfp`; bidders
bid; the winner provisions a guest **via cloud-init `user_data`**
(`buildUserData` composer + module registry in `cloud-init-common`); the requester reaches the guest
**only through the relay** (ssh `ProxyCommand` over the websocket tunnel). The
relay is the registry. Nothing talks to a guest except through this path.

**NEVER — non-negotiable, breaks the whole point of the project:**

- Provision a container/VM by hand in code OR tests (`container run`,
  `docker run`, `container exec ... apt-get install`, mounting a binary,
  `ssh-keygen` + manual `authorized_keys`). Guests are born from cloud-init
  `user_data` produced by the RFP flow, never hand-assembled.
- Cross-compile/mount a guest agent to skip cloud-init. The agent (sshd,
  websocat, fedproxy-client, tunnel subscriber, …) is installed BY the
  cloud-init `user_data` the bidder applies.
- Write a "real ssh / real guest" e2e that stands up its own container instead
  of driving `runComputeContract(...)`. If a test needs a live guest, it drives
  the requester (`runComputeContract`) against a real local bidder running a
  container-mode compute provider (see
  `atproto-market/test/bidder_container_integration_test.ts` for the canonical
  harness: local dispatcher + fake PLC + bidder + requester, fetch patched so
  `https://*.localhost` → local dispatcher). Provisioning ALWAYS goes
  requester → RFP → bid → accept → cloud-init.
- Add a second SSH/tunnel transport that the RFP cloud-init does not deploy. New
  guest-side transport = add a `UserDataModule` to `cloud-init-common`'s registry
  (or a sibling user_data builder) that installs it, so the RFP flow remains the
  only way a guest comes up.

**Pre-flight before any provisioning / guest / ssh / tunnel code or test:** Does
this go through `runComputeContract` + RFP records + cloud-init? If it calls
`container run`/`docker run`/`exec` directly, STOP — it is wrong.

## Bash CWD

Bash tool persists CWD across calls. `cd` in one call = CWD for next call.
Always start every Bash command with `cd <absolute-path> &&` when targeting
a specific workspace. Never assume CWD.

```
cd /home/johnandersen777/src/publicdomainrelay/hono-pds && deno test ...
cd /home/johnandersen777/src/publicdomainrelay/hono-compute-provider && deno check ...
```

## Layers

Every capability ("concept") split 4 ways. Path shows layer:

```
lib/common/${shared}                       leaf utils, types, constants (no concept logic)
lib/abc/${concept}                         interfaces + pure state, zero I/O
lib/${concept}-${transport}                impl: timers, crypto, fetch, sockets
lib/hono-factory-${concept}-${transport}   final Hono integration, composed not subclassed
hono-${concept}                            thin CLI: read config, build factory, serve
```

Deps flow ONE way. No cycles:

```
lib/common/*                  <- external deps ONLY
   ^
lib/abc/*                     <- imports common (type imports)
   ^
lib/${concept}-${transport}   <- imports abc + common
   ^
lib/hono-factory-*            <- imports impl + abc + common + hono
   ^
hono-* (CLI)                  <- imports hono-factory + common + external
```

Org-root `typescript-helpers/` holds cross-repo shared utils (logger, event bus,
cli-args-env, http-error, hono-error-middleware, hono-factory-static-files-fs).
Check there BEFORE writing any new `lib/common/` package. Two repos need it = it
belongs there.

## PATTERNS — ALWAYS DO

- Structured log port via `onListen` when using `Deno.serve`

## ANTI-PATTERNS — NEVER DO

Non-negotiable. Each breaks architecture. Detail + examples below in per-layer
sections.

**Structural**
- Sub-module exports (`exports: { "./sub": ... }`). One package = one `mod.ts`.
- Cross-concept imports (`relayer` imports `subscriber`). Shared -> `lib/common`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [publicdomainrelay/socialweb-computer](https://github.com/publicdomainrelay/socialweb-computer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->

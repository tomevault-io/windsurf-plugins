---
trigger: always_on
description: > **The rename to `reemoat` is complete, and every `remoslop` left in this tree
---

# Reemoat

> **The rename to `reemoat` is complete, and every `remoslop` left in this tree
> names something that is not this product.** One of them is load-bearing:
> `LEGACY_STORAGE` in `packages/web/src/cp.ts` holds `remoslop.credential` and
> `remoslop.apiKey`, which name what is **already sitting in somebody's browser**,
> so that a rename does not sign the fleet out. They are read once, adopted and
> swept, and they get deleted rather than updated once no tab has been signed in
> since before the rename — `webcheck` pins both halves. A blanket rename caught
> them once and the failure was total: `setSession` writes the credential and then
> sweeps that list, so with the same string in both it deleted what it had just
> written. The rest are history rather than behaviour: Q7.71 records what the
> migration had to move by hand, and a few comments quote measured paths from the
> machine this was developed on (`/Users/rends/remoslop…`), which are still the
> paths that were measured. **None of them may be swept by a rename.**


A daemon that owns coding-agent sessions and exposes them over HTTP + WS, a
control plane that issues identity and relays every request to them, and a web UI
that supervises all of it from a phone.

**One person, one machine, many agents, and no sandbox.** The daemon runs on your
own machine and spawns agents as children of itself, as you — the same thing that
happens when you type `claude` in a terminal, except that you can be somewhere
else. Several at once, each in its own git worktree, each able to bring up a dev
server and run what it just wrote. Multi-user moved to the control plane: several
people, each with their own machine and a grant on it. The daemon accepts any
token whose `aud` is its own machine id and stops asking who the subject is.

It spawns `claude`, `kimi` or `codex` over ACP (Agent Client Protocol), normalizes
all three into one event union, and puts that behind a network layer built on the
assumption that **clients are unreliable**: a laptop lid closes, a phone drops to
LTE, a tab is discarded. The daemon is the source of truth and the agent must
never notice a client leaving.

Node >= 24, ESM, TypeScript strict. Everything in `src/`, `scripts/` and
`packages/control-plane` runs straight off `tsx` with no build step;
`packages/web` is bundled by Vite, inside the control plane's image — the only
thing here that compiles anything.

**No test framework.** `typecheck`, `authcheck`, `daemoncheck`, `relaycheck`,
`webcheck`, `pincheck`, `deploycheck`, `docscheck`, `imagecheck` and `harness`
are the whole automated safety net, and they are drivers rather than unit tests
on purpose. Eight run offline in one process with no fleet, no agent and no
deploy — `docscheck` is the newest and the only one whose subject is prose: it
holds this file to a budget, because the last time it was cut nothing checked
the result and it was larger six days later.
`harness` drives a real agent and needs a login CI cannot hold. `imagecheck`
builds and starts a container, so it is a separate CI job — and it earns that:
the control plane reaches into the repository root for a file list written down
**twice**, in `.dockerignore` and in `deploy/docker/Dockerfile`'s COPY lines, and
an import missing from either passes `typecheck` and all seven other drivers while
breaking only the image. Measured while adding `src/http.ts`: missing from
`.dockerignore` it fails at COPY with `"/src/http.ts": not found` (the build
context never carried it), and missing from the Dockerfile it fails later with
`Cannot find module`. Adding an import means editing both.

Deploying is a *separate* act from checking, and nothing does it on a push.

> **Why any of this is the way it is lives in `docs/DECISIONS.md`** — 730 entries
> as question → decision, with the measurement behind each and the alternatives
> that were tried and taken back out. **The count is asserted by `docscheck`
> rather than restated here from memory**, which is the whole reason it is right:
> it said 453, then 294, because it was re-derived by hand from `### Q…` alone
> while Q3 and Q5 — the two largest groups — sit at `####`. This file states rules
> as they stand and names the symbol that enforces each; that one answers *why*,
> and is where to look before reversing anything.

## Commands

```bash
pnpm typecheck                       # tsc --noEmit, both packages
pnpm authcheck                       # token verification and enrollment
pnpm daemoncheck                     # the daemon's HTTP surface and durable state: routes,
                                     #   the v6 migration, the login pty, the WS, subagent lineage,
                                     #   permissions, stopping a turn, the SQLite log, changes/diff,
                                     #   uploads — and the bounds an agent can push against,
                                     #   all of them refusals now: a permission's title and
                                     #   options weighed as one 8 KiB thing rather than clipped,
                                     #   a form's prose carried whole against one 32 KiB
                                     #   backstop, `locations` in the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rends-east/reemoat](https://github.com/rends-east/reemoat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->

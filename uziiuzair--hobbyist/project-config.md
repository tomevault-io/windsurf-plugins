---
trigger: always_on
description: Read this first, every session. It is the standing context for the whole repo.
---

# Hobbyist

Read this first, every session. It is the standing context for the whole repo.

## What this is

**A self-hosted platform that feels like Neon and Supabase, on hardware you own.**

Managed platforms sell convenience: instant provisioning, branching, scale to
zero, a connection string that just works, a dashboard that makes a database
legible, a deploy that happens on push. Every primitive underneath that
convenience is already free and mature. The convenience itself is what people
pay for, and it is the only thing missing from the self-hosted world.

Hobbyist is that convenience layer. One command gives you a project with a
Postgres in it. A studio you would actually choose to open. Everything sleeps
when nothing is using it and wakes when something connects. It runs on one box,
with no Kubernetes, and you can walk away from it at any time with a single
command.

**One-liner:** Your stack. Your box. Their convenience.

## The wedge

**Everything sleeps, and everything wakes on demand.**

This is the single reason for the project to exist, and every design decision
serves it. It is also the one thing no self-hostable alternative does:

- **Self-hosted Supabase never sleeps.** It is a large multi-container stack that
  runs at full cost whether or not anything is using it.
- **Xata's open-source scale-to-zero plugin cannot wake a database.** In their
  own words, it "can't handle reactivation because the cluster is no longer
  running once it's hibernated." Automatic reactivation on connection is what
  they kept in the paid cloud.
- **Coolify and Dokploy deploy apps well and do not sleep them.**

Sleep is what makes ten projects fit on one small box. Wake is what makes sleep
invisible. Neither is useful without the other, and the pair is the product.

When a feature conflicts with the wedge, the wedge wins.

## What this is not

This is **not a business.** Nobody is expected to pay. There is no cloud offering,
no hosted tier, no paid feature, no metering, no billing, and no roadmap toward
one. Nothing is held back behind a paid guard. The product exists so that people
who want this option have it, and we are happy knowing it helps people.

**We provide no warranty and take on no liability.** That is stated plainly in
the licence and it is not a throwaway. It is why we do not ship anything whose
failure mode is someone else getting owned or losing data quietly.

That not-a-business fact is load-bearing, not a disclaimer. It removes
multi-tenancy, isolation hardening, usage accounting, and quota enforcement from
scope, and those removals are what make the project buildable by one person.

**Success metric:** the author is still using it daily in six months. That is the
only one. Stars, forks and issue count are noise.

## Assets

| Asset | Value | Status |
|---|---|---|
| Domain | `hobbyist.sh` | Owned |
| NPM namespace | `@hobby.sh/*` | Owned. Registry showed the scope unclaimed as of 2026-08-06 |
| Repository | `github.com/uziiuzair/hobbyist` | Live, `main` |
| CLI binary | `hobby` | The primary user-facing entry point |
| Parent | Ooozzy Ltd (product studio) | This is studio infrastructure, not a studio product |

Six packages under the namespace:

```
@hobby.sh/core     Project and Resource model, state, config, ComputeRuntime interface
@hobby.sh/pg       the postgres resource kind: lifecycle, data directories, readiness
@hobby.sh/proxy    the wake router. Postgres wire protocol, HTTP from Phase 2
@hobby.sh/cli      the hobby binary and the daemon
@hobby.sh/studio   the web UI and its auth
@hobby.sh/mcp      MCP tools over the daemon API
```

Small, composable, independently versioned. A package that cannot be used without
three siblings is a module, not a package.

## Direction

The stance the whole project serves: **developer ownership over platform
dependence.** Infrastructure should be something you can leave.

Concretely, three promises, in priority order when they conflict:

1. **You can always leave.** The data directory is a plain Postgres data
   directory. `pg_dump` always works. `hobby eject` hands you a
   `docker-compose.yml` and the data, and gets out of the way.
2. **It runs on one box.** A five dollar VPS, a Mac Mini, an old ThinkPad under a
   desk. If a feature requires a cluster, it is out of scope.
3. **It feels good.** The reason people pay for managed platforms is ergonomics.
   Matching the primitives is not enough; matching the feel is the entire job.

## Scope

A **Project** is a namespace holding typed resources. Phase 1 registers one kind,
`postgres`. Later phases register more by implementing an interface, and earlier
phases do not change. That shape is the reason the phases below are additive
rather than successive rewrites. See ADR 0007.

| Phase | Ships |
|---|---|
| **1** | Studio, Postgres, CLI, MCP |
| **1.5** | Copy-on-write branching |
| **2** | Compute: workers and apps, stateless, arbitrary runtimes via containers |
| **3** | S3-compatible object storage, volumes for compute, React SDK |

Phase 2 compute is deliberately **stateless**. It gets its persistence from
Postgres. Volumes arrive in Phase 3, which keeps volume lifecycle out of the
hardest phase.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [uziiuzair/hobbyist](https://github.com/uziiuzair/hobbyist) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->

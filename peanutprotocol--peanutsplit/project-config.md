---
trigger: always_on
description: This repo does **not** follow mono's always-PR workflow. Split is growth-owned, deliberately small, and meant to be shippable (and killable) in an afternoon. `mono/CONTRIBUTING.md` governs `peanut-ui` and `peanut-api-ts`; the rules below govern this repo and win where they differ.
---

# Working rules — peanutsplit

This repo does **not** follow mono's always-PR workflow. Split is growth-owned, deliberately small, and meant to be shippable (and killable) in an afternoon. `mono/CONTRIBUTING.md` governs `peanut-ui` and `peanut-api-ts`; the rules below govern this repo and win where they differ.

Product rationale, status, milestones and the decision log live in the Notion project (linked from `mono/projects/peanut-split/`). The engineering backlog — what's built, building, queued, gated on infra, and deliberately not built — lives in [`ROADMAP.md`](ROADMAP.md). This file is the working rules only — don't restate either here.

## Readability for all work

- Apply this standard to user-facing copy, docs, comments, identifiers, PR and commit explanations, and assistant responses.
- Lead with the concrete point or action. Use familiar words and complete sentences in prose; keep labels and identifiers concise.
- Cut padding, inflated claims, forced personas, rhetorical questions followed by answers, and invented contrasts.
- Keep the facts, technical precision, necessary detail and natural phrasing in each language.
- Name code for what it represents or does, using existing domain terms consistently.
- Use comments to explain a reason or constraint. Omit comments that merely narrate the code.
- Choose code that is easy to follow. Add abstractions only when they reduce actual complexity.
- Before delivery, do a final editorial reread from the human reader's perspective; pattern checks alone are insufficient. Preserve factual and functional meaning when editing.

## What's in here

```
apps/api    Fastify + Prisma + its own Postgres. All money logic: splits,
            balances, FX, settlements, and the settle-with-Peanut loop.
apps/web    The live product — Next, PWA, per-room link previews. THIS is what
            peanutsplit.com serves.
```

`main` is canonical: two apps, one of them live, nothing dead. The original
standalone UI (the settle-up screens wired to `apps/api`'s settle loop) lives on
the **`poc/original-split-ui`** branch — pull the screens from there when porting
them into `apps/web`, don't develop on it.

One seam still open: `apps/web` is outside the pnpm workspace (it brought its own
lockfile) and talks to its own database rather than `apps/api`. Collapsing those
two is the remaining merge.

## Shipping

- **Push straight to `main`.** No PR, no review gate, no waiting. `main` is unprotected on purpose.
- **A push to `main` deploys to production within ~5 minutes.** There is no CI gate in front of it. Run the checks below yourself.
- Open a PR only when you actually want a second opinion, not as ceremony.
- Commit messages explain _why_. No AI co-author lines.

## What still holds

- **Money code needs a test before it ships.** Balances, splits, FX, settlements: if it can produce a wrong number, it has a test. The pure math in `apps/api/src/split/math.ts` is where that logic belongs — testable without a database.
- **The money surface is frozen.** Split settles through an existing Peanut payment-request link. No new money-path endpoints. Split is accountless: recent rooms and member tokens stay on the device, and the room link stays the credential. No email login, passwords, OAuth, profiles, or room ownership. Push notifications are opt-in per device per room. Anything past that is a product decision, not an implementation gap.
- **Do not scaffold deferred ideas.** Future-facing fields, states, routes, or abstractions require an approved ROADMAP item that explicitly authorizes implementation; listing an idea as deferred does not.
- **No identity in analytics either** — no room slug, no member name, no amount. The slug is the room's access control; a name is what someone chose to show their friends. Rooms are grouped by `analyticsKey`, a one-way digest of the room UUID issued by the server — it cannot be turned back into a room, and the slug still never leaves the device. Attach it with `roomProps(slug, …)`, which looks the key up; never put a slug in a property bag.
- Dependencies must be ≥14 days old (`.npmrc` enforces it).
- Run `pnpm typecheck && pnpm test && pnpm format` before pushing. Run `pnpm format` from `apps/web`, never the repo root — the root config can't resolve `prettier-plugin-tailwindcss` and reformats CI yaml as a side effect.

## Local dev

```bash
pnpm bootstrap    # NOT plain `pnpm install` — see below
pnpm dev          # API :5051 + web :3000 (or dev:api / dev:web for one)
```

`apps/web` is not a workspace member, which has one sharp edge: running
`pnpm install` inside `apps/web` walks _up_ and installs the workspace instead,
leaving `apps/web/node_modules` missing. It needs `--ignore-workspace`, which is
what `pnpm bootstrap` does. (Docker doesn't hit this — the web image's build context
is `apps/web` alone, so there's no parent workspace file to find.) The root
scripts reach it with `--dir apps/web` rather than a filter. If you add an app, wire it into the root

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [peanutprotocol/peanutsplit](https://github.com/peanutprotocol/peanutsplit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->

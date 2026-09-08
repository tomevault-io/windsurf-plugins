---
trigger: always_on
description: Read at the start of every session. **This file is a router, not an encyclopedia.** It holds
---

# TikTok-Clone — Agent Briefing

Read at the start of every session. **This file is a router, not an encyclopedia.** It holds
only what applies to every session; everything narrower lives one level down and loads on
demand.

A TikTok-style short-form video Android app (Kotlin, Firebase backend, Clean Architecture +
Jetpack Compose — the Fragment/LiveData/Groupie → Compose migration completed 2026-08-09).

## 1. Where knowledge lives

Do not guess, and do not ask the user for something in this table. Go read it.

| Question | Source |
| --- | --- |
| What does the product do, what are the entities, what's the architecture? | [docs/SPEC.md](docs/SPEC.md) |
| How far has the Clean Architecture/Compose migration gotten? | [docs/CLEAN_ARCHITECTURE_MIGRATION.md](docs/CLEAN_ARCHITECTURE_MIGRATION.md) |
| I'm working on onboarding/auth — what's the contract and which pass am I in? | [docs/plans/ONBOARDING_FEATURE_SPEC.md](docs/plans/ONBOARDING_FEATURE_SPEC.md), then the matching prompt in [docs/plans/prompts/](docs/plans/prompts/) |
| Gotchas for the layer I'm editing (`domain`/`presentation`/`repo`) | The nearest `AGENTS.md` (see §2) |
| How do I build/run/test this locally? | §7 below |
| Why does this code look like this? | `git log -p` on the file, then `docs/SPEC.md` §7 for known-dead integrations |

**Precedence when sources disagree:** `docs/SPEC.md` > nested `AGENTS.md` > `README.md`.
`README.md` is user-facing and already has known drift from actual build config (tracked in
`docs/SPEC.md` §2) — don't treat it as a source of engineering truth. A disagreement between
`docs/SPEC.md` and the actual code is a P0: stop and reconcile with the user before writing
more code on top of it.

## 2. Nested AGENTS.md — read the one for your area

| Working in | Read |
| --- | --- |
| `app/src/main/java/com/andre_max/tiktokclone/domain/**` — use cases, `Result`/`DataError` | [domain/AGENTS.md](app/src/main/java/com/andre_max/tiktokclone/domain/AGENTS.md) |
| `app/src/main/java/com/andre_max/tiktokclone/presentation/**` — Fragments, ViewModels, Compose screens | [presentation/AGENTS.md](app/src/main/java/com/andre_max/tiktokclone/presentation/AGENTS.md) |
| `app/src/main/java/com/andre_max/tiktokclone/repo/**` — Firebase/Room/device data sources | [repo/AGENTS.md](app/src/main/java/com/andre_max/tiktokclone/repo/AGENTS.md) |

If you're about to touch one of those trees and haven't read its file, read it first. It
exists because someone already made the mistake you're about to make.

## 3. Reach for the right mechanism

| Situation | Use | Why |
| --- | --- | --- |
| Android SDK management, running/screenshotting the app on a device or emulator, Android API docs | `android-cli` skill | Already installed globally; covers `sdk`, `create`, `run`, `emulator`, `screenshot`, `docs` |
| Broad search where only the conclusion matters ("where is X handled across the app") | `Explore` subagent | Keeps the main context clean; don't use it for a lookup you can `grep` directly |
| A specific symbol, string, or file you can name | `grep`/`rg` directly | One tool call, near-zero tokens — see the global search-efficiency guidance already loaded for every session |
| Any shell command | Runs through the global `rtk` proxy hook automatically | Transparent token savings, no action needed from you |
| Spawning an Agent tool call | Guarded by the global `grep-before-agent-guard` hook | If it blocks you, it's telling you a grep would've been cheaper — try that first |

**The rule that decides:** if a script can determine the answer, it's a hook, not prose in
this file. If it needs judgment, it's a skill or a spec. Only project-wide standing context
belongs here.

## 4. Non-negotiables

Full detail and rationale in [docs/SPEC.md](docs/SPEC.md) §9. One line each:

1. **App compiles and runs after every step** — never leave it broken between sessions.
2. **Old and new systems coexist during migration** (strangler-fig) until fully unreferenced,
   then deleted in one dedicated step — never delete legacy code as a drive-by.
3. **Tests first for anything with logic**, especially use cases.
4. **One feature vertical slice at a time**, not one layer across every feature.
5. **Grep for and read the current implementation before replacing business logic** — never
   assume a "textbook" rule matches what's actually enforced today.
6. **Dependencies point inward**: `domain` imports nothing from `repo`/`presentation`. Proof:
   `grep -rn "^import" app/src/main/java/com/andre_max/tiktokclone/domain | grep -Ei "android|firebase"` must return nothing.

## 5. Abstraction: on evidence, not on anticipation

Abstract now, even at one caller, when it marks a boundary the architecture requires (a repo
interface keeping `domain` pure, a `Result`/`DataError` type enforcing typed errors at the
data boundary) or names a domain concept `docs/SPEC.md` already names. Don't abstract for
hypothetical reuse — duplicate it; the third occurrence is the signal. A boolean parameter
added so an existing function can serve a second case is the wrong abstraction announcing
itself — write the second function instead.

## 6. Working style

- Ask before making an architectural or behavioral call that isn't already pinned down in

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DaChelimo/TikTok-Clone](https://github.com/DaChelimo/TikTok-Clone) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->

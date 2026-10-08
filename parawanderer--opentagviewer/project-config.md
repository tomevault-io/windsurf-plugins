---
trigger: always_on
description: Rules for agents working on OpenTagViewer. Written for automated contributors, but the
---

# AGENTS.md

Rules for agents working on OpenTagViewer. Written for automated contributors, but the
constraints apply to anyone.

**This file is owned and maintained by agents, and keeping it current is part of the work rather
than a favour to whoever comes next.** @parawanderer asked for that and has not read it since. No
human reviews what lands here, so the agent writing a change is the only check on it.

What that means in practice:

- **If you learned something the hard way, write it down in the same change that taught you.** The
  next agent starts here, and anything left out is an afternoon they pay for and you did not.
- **The reason travels with the rule.** An instruction with no reasoning attached is one a later
  agent deletes the first time it looks inconvenient. That is why the entries here argue.
- **Earn it.** Nobody is filtering this, so a rule that has not actually cost somebody something
  does not belong. Rule 10 has the test.
- **Do not carry this register outside the repository.** It is written for agents who need to be
  argued out of breaking something. A pull request review, an issue reply, or anything else a
  person reads is a different job. See rule 16.

## What this project is

An Android app that shows Apple AirTag locations without an Apple device. Tags are exported
from a Mac once (see the wiki), imported as a zip, and located via Apple's Find My network
through [FindMy.py](https://github.com/malmeloo/FindMy.py).

| Layer | Technology |
| --- | --- |
| App | Android, **Java** (not Kotlin), Gradle Kotlin DSL, `compileSdk`/`targetSdk` 35, `minSdk` 24 |
| Apple protocol | **Chaquopy** — CPython 3.12 embedded, bridged in `app/src/main/python/main.py` |
| Persistence | **Room** (`opentagviewer-db`) |
| Async | **RxJava3** through repositories and services |
| Boilerplate | **Lombok** (`@Builder`, `@Getter` on entities) |
| Desktop exporter | `python/` — tkinter wizard, PyInstaller |

`app/src/main/python/` is packaged into the APK by Chaquopy. Nothing in it may import
Android or Java types — that is what makes it testable on plain CPython.

---

## Rules

### 1. Never ship a Room schema change without a migration

Bumping `version` without a `Migration` throws at first database access **on upgrade only**.
A fresh install works fine, so it passes casual testing and then destroys every existing
user's imported beacons and location history.

- Add a `Migration` and register it in `addMigrations(...)`
- **Never** use `fallbackToDestructiveMigration()` — recovery means redoing the macOS
  export, and `allowBackup` is false so there is no backup
- Add a migration test, including the direct path from the oldest version (users skip
  releases)
- Schemas are exported to `app/schemas/` and committed; include the new JSON in your change

### 2. Do not claim something works if you have not run it

Say plainly what was verified, what was assumed, and what could not be checked. If a change
needs a real Apple account, a real Mac, or hardware you do not have, state that rather than
implying coverage. A wrong "this works" is worse here than an honest gap, because the
failure mode is silent data loss on other people's devices.

### 3. Verify library behaviour against the installed version

FindMy.py's API changes significantly between minor versions. Methods and properties that
existed in 0.7.x are gone in 0.9.x. Before "fixing" code that looks wrong, check the pinned
version's actual source — several apparent bugs here are forced adaptations.

```bash
python -m venv .venv && .venv/bin/pip install "FindMy==<pinned version>"
```

### 4. Respect the Anisette constraints

- **FindMy's own local provider still does not work.** `anisette` needs `unicorn`, a CPU
  emulator Chaquopy cannot build for Android. `app/stubs/unicorn/` is a stand-in that makes
  the dependency tree resolve; every method raises. The stub is not a step toward enabling
  `aniLocal`, and never will be.
- **The app does run ADI locally, by a different route.** `app/src/main/java/.../anisette/`
  loads Apple's real Android ADI libraries in-process and produces Anisette here, falling
  back to a remote server when it cannot. That is unrelated to `aniLocal` and needs no
  emulator — the libraries are native Android code. See `CONTRIBUTING.md` for how to run its
  tests.
- **Sessions are bound to one machine identity.** Local and remote Anisette present different
  ones, as do two different servers. Changing it requires a re-login. That is inherent to how
  Apple binds the session, not a bug to work around.
- **So a fallback must say so.** Accounts are deliberately serialized as `aniRemote` even
  when established locally, because the ADI state lives in app storage and not in the account
  — this keeps exported logins restorable anywhere. The cost is that a locally-established
  session can end up continuing against a server, and Apple will see a different machine.
  `LocalAnisette.recordSessionProvenance` records which kind established the session so that
  this is reported rather than presenting as auth that silently stops working. Never remove
  that warning to make a log quieter.

### 5. Never bundle an AMap API key


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [parawanderer/OpenTagViewer](https://github.com/parawanderer/OpenTagViewer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->

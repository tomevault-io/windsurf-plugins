---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

FactoriOS is a joke Linux distribution whose sole purpose is to boot into Factorio. Treat the joke premise as a real design constraint: the system should do nothing else of note. Don't add general-purpose desktop features or "useful" extras — if it isn't in service of launching and running Factorio, it doesn't belong.

## Architecture

Three runtime layers, each in its own top-level directory:

- **`launcher/`** — pure Python library. All factorio.com interaction (api-login token auth, version download, archive extraction), and all on-disk layout knowledge (versions, profiles, the `last-user` pointer). No UI. Both the greeter and any future CLI tooling depend on it.
- **`greeter/`** — GTK4 Python app. The *only* user-facing surface on an installed system. Login screen → version/profile chooser → spawns Factorio as a child → returns to chooser on game exit. All HTTP runs on a worker thread; results posted back via `GLib.idle_add`.
- **`iso/` + `installer/`** — the install-time path. archiso profile boots a live env that autoruns the shell installer; the installer partitions, pacstraps Arch base + the `factorios-*` packages, and enables `factorios.service`.

Glue: `systemd/factorios.service` runs `/usr/bin/factorios-session` on tty1 as the `factorios` user (UID 1000). The wrapper logs the full session transcript to `/tmp/factorios-session.log` (env, `/dev/dri/` state, labwc `-d` debug, greeter stderr, Factorio stderr) and then exec's `labwc -d -s /usr/bin/factorios-greeter`. There is no display manager and no Unix-level login — factorio.com identity is the only identity.

## Identity model (load-bearing)

No local Linux user accounts are ever created. One shared system user runs sessions; per-user state is segregated by directory under `/var/lib/factorios/users/<factorio-username>/`. Authentication is purely against factorio.com. If you find yourself reaching for `useradd`, PAM, or per-user homes, stop — that's the wrong layer.

## Filesystem layout (installed system)

```
/var/lib/factorios/
  versions/
    <version>-<build>/             # build ∈ {vanilla, space-age}; shared
    _demo/                         # demo install; no build dimension
  users/<factorio-username>/
    profiles/<build>/<name>/       # full Factorio write-data dir per
                                   # profile — saves/, mods/, config/,
                                   # player-data.json, achievements.dat …
    session.json                   # {"username": ..., "token": ...}
    last-launch.json               # remembered (build, version, profile)
  users/_guest/
    profiles/<n>/                  # guest/demo profiles (flat — no build dim)
  last-user                        # present iff Remember Me
```

`launcher/factorios_launcher/paths.py` is the single source of truth — don't hard-code these elsewhere. Build identifiers are `vanilla` / `space-age` user-facing; the factorio.com download API uses `alpha` / `expansion` — the mapping lives in `paths.BUILD_API` and the rest of the codebase never deals in alpha/expansion. Profiles are per-build because mod compatibility differs across the two; the guest/demo flow is flat (no build dim) because the demo is its own build.

**A profile is a complete Factorio write-data dir.** Before launch, `profiles._link_home_factorio()` symlinks `~/.factorio` at `users/<u>/profiles/<build>/<name>/`, so saves, mods, config, `player-data.json`, achievements, and everything else Factorio writes is scoped to the chosen profile. Switching profiles re-points the symlink; the only Factorio CLI flag we pass is `--config /home/factorios/.factorio/config/config.ini`, which reaches the same profile through the symlink. (We tried `--write-data` — not universally recognised — and `--mod-directory` — works but redundant once the whole profile is `~/.factorio`. Service credentials are JSON fields in `player-data.json`, not CLI args.) A one-shot migration on first launch folds any pre-existing `users/<u>/factorio/` content into whichever profile launches first.

Space Age ownership lives on `Session.has_space_age` (refreshed on every `login()` / `validate()` via a HEAD probe against the expansion download endpoint, never persisted). The chooser hides the Build selector when `has_space_age == False`.

User + dir ownership are declarative: `/usr/lib/sysusers.d/factorios.conf` creates the `factorios` user (UID 1000) with seat/video/input/render group memberships; `/usr/lib/tmpfiles.d/factorios.conf` enforces ownership of `/var/lib/factorios/{,versions,users}` on every boot. Don't put `useradd` or `chown` in `.install` hooks — it's the wrong layer and `useradd --groups` fails the whole user creation if any one group is missing.

## factorio.com auth flow


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tomribbens/FactoriOS](https://github.com/tomribbens/FactoriOS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->

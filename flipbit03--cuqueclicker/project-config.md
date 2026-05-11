---
trigger: always_on
description: A TUI parody of Cookie Clicker — you finger an ASCII anus instead of clicking a cookie. Bilingual pun: in Brazilian Portuguese "Cookie Clicker" ≈ "Cu que clicker" ("the ass that clicks"). The Portuguese framing is the whole joke; keep it in mind when making copy/naming calls.
---

# CuqueClicker

A TUI parody of Cookie Clicker — you finger an ASCII anus instead of clicking a cookie. Bilingual pun: in Brazilian Portuguese "Cookie Clicker" ≈ "Cu que clicker" ("the ass that clicks"). The Portuguese framing is the whole joke; keep it in mind when making copy/naming calls.

Written in Rust with `ratatui` + `crossterm`. Runs as a single self-contained binary on Linux (x86_64 & aarch64 musl), macOS (aarch64), and Windows (x86_64 MSVC, static CRT). Shipped via GitHub Releases + crates.io.

## Audience & tone

- **README tone is "halfway-crude"**: name the bit (parody, ass, finger a cuque, Portuguese pun) explicitly but don't lean on shock value. Technical sections (Install/Controls/License) stay plain and professional.
- **Public docs are English-only.** pt_BR lives inside the game (auto-detected from `$LANG`); it does not leak into README/CLI help. Don't leave Portuguese words ("Papel de Seda", "Prestígio") in the EN strings — translate them. Each locale is internally consistent.

## Project policies (specific to this repo)

- **Saves must always load cleanly across versions — never break or lose a user's savestate.** Any schema change (renamed field, added field, reordered variant, inserted tier, new serialized sub-state) must be paired with explicit migration code in `GameState::migrate()` that rewrites old saves into the new shape. Design every change with this in mind *before* touching the struct. `serde` aliases and leave-the-old-name-around shims are not the mechanism — do clean renames and absorb the cost in `migrate()`.
- **Catalog state (fingerers / upgrades / achievements) is addressed by stable string IDs, not positional indices.** `GameState` stores `fingerers_owned: HashMap<String, u32>`, `upgrades_earned: HashSet<String>`, `achievements_earned: HashSet<String>`. Each `FINGERERS`/`UPGRADES`/`ACHIEVEMENTS` entry has an `id: &'static str` that MUST stay stable across the lifetime of the game — renaming an id silently orphans every player's progress on that item. Reordering, inserting, or removing entries in these tables is free: unknown ids in a save are ignored (forward-compat), missing ids default to zero/absent (backward-compat). New content = add catalog entries; retired content = remove them. Neither requires a migration.
- **Every migration branch must ship with a unit test.** `#[cfg(test)]` module lives next to `migrate()` in `src/game/state.rs`. Each test constructs an old-shape `GameState` (or deserializes an old JSON fixture), runs `migrate()`, and asserts the resulting counts, ids, and invariants are what the live game expects. Never commit a migration without the test that proves it.
- **MIT licensed.** `LICENSE` carries the standard MIT text; `Cargo.toml` uses `license = "MIT"` (SPDX). If someone wants to fork / vendor / adapt, that's fine.
- **No CHANGELOG.md.** Rely on git log + GitHub Release notes.
- **No code-signing / notarization.** Windows users click through SmartScreen; macOS users run `xattr -d com.apple.quarantine` or right-click→Open. Do not add signing steps without explicit ask.
- **No backward-compat hacks in general** — delete dead code rather than keeping `// removed` markers, don't rename `_unused` vars, don't re-export removed types. (User's global convention, reinforced here.)

## Dev vs release: the two gates

Two independent mechanisms decide whether a binary is "dev":

1. **`Cargo.toml` `version = "0.0.0"`** — pinned in the repo. `release.yml` `sed`-patches it from the git tag at build time. Anything compiled from an unpatched tree reports `0.0.0`.
2. **`build_info::is_dev_build()`** = `VERSION == "0.0.0"`. This, AND NOT the `binary-release` cargo feature, gates dev-only surfaces.

Dev-only surfaces (all require `is_dev_build()`):
- **Debug pane** (F1 / F2 / F3 spawn Lucky/Frenzy/Buff goldens; F4 gives free cuques). Also requires `!cli.no_debug`. `--no-debug` is opt-**out** on dev; it's labelled "disable" not "hide" because the point is that the cheats are gone, not just invisible.
- **`--demo-for-recording [SECONDS]`** (hidden from `--help`). Runs the auto-driver on an ephemeral rich state.

The `binary-release` cargo feature is a **different** gate — it only toggles how `cuqueclicker self update` re-installs:
- Feature ON (set by `release.yml`) → re-run the installer script (curl+sh / irm+iex).
- Feature OFF (local build, `cargo install`) → `cargo install cuqueclicker --force`.

HUD shows `v0.0.0 (dev)` in dev, plain `vX.Y.Z` in release. The title is built in `src/ui/mod.rs` from `env!("CARGO_PKG_VERSION")`; `i18n::title` is just the bare name.

## Invariants — don't casually break these

- **Catalog `id` strings are load-bearing forever.** Once a `FingererStats`, `UpgradeKind`, or `AchievementKind` ships with an id, renaming that id silently zeroes every existing save's progress on that item. Treat ids like primary keys. Cosmetic names are in i18n and can change freely; ids stay.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [flipbit03/cuqueclicker](https://github.com/flipbit03/cuqueclicker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->

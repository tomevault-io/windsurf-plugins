---
trigger: always_on
description: Fono is a GPL-3.0 Rust single-binary voice dictation tool for the desktop. It replaces
---

# AGENTS.md — Fono Agent Orientation

## What is Fono?

Fono is a GPL-3.0 Rust single-binary voice dictation tool for the desktop. It replaces
[Tambourine](https://github.com/kstonekuan/tambourine-voice) (Tauri + Python) and
[OpenWhispr](https://github.com/OpenWhispr/openwhispr) (Electron) with a lighter native
stack — no WebKit, no Node, no Python — while unioning their feature sets (global hotkey
push-to-talk, local + cloud STT, optional polish, text injection, tray UI, history).
Target users: Linux desktop (i3 / sway / KDE / GNOME, X11 and Wayland), Windows, and macOS.

## Orientation: read in this order

1. `docs/plans/2026-04-24-fono-design-v1.md` — authoritative design and 10-phase
   implementation plan. This is the source of truth for *what to build and when*.
2. `docs/decisions/` — Architecture Decision Records (ADRs) explaining *why* key
   choices were made (language, name, license, default models).
3. `docs/status.md` — current phase, what's next, session log.
4. `CONTRIBUTING.md` — DCO sign-off requirement, formatting, and clippy rules.

## Current phase

**Phase 0 complete; Phase 1 next.** See `docs/status.md` for details.

## External references

- `/mnt/nvme0n1p5/Work/slackbuilds/earlyoom/` — NimbleX SlackBuild template to mirror
  when Phase 9 packaging lands.
- `/mnt/nvme0n1p5/Work/slackbuilds/tambourine-voice/` — earlier aborted attempt to
  package Tambourine on NimbleX; catalogued missing system deps (webkit2gtk-4.1,
  python3.13, uv, libxdo, libayatana-appindicator3). Useful reference for Phase 9
  dependency negotiation.
- <https://github.com/kstonekuan/tambourine-voice> and
  <https://github.com/OpenWhispr/openwhispr> — upstream projects whose feature union
  Fono is replicating.

## Hard rules for agent sessions

- **Pre-commit gate (run, in order, before EVERY `git commit` and EVERY
  `git push`):**
  1. `cargo fmt --all -- --check` — must exit 0. If it fails, run
     `cargo fmt --all` and re-stage. Do **not** push fmt-dirty code; CI
     will reject it at the `cargo fmt --check` step (see
     `.github/workflows/ci.yml`). This caught us once on commit
     `33e3e51` — never again.
  2. `cargo clippy --workspace --all-targets -- -D warnings` — must
     exit 0. Same lint set as CI; passes locally ⇒ passes there. If CI
     stops at fmt it will *not* surface clippy errors, so always run
     clippy locally too.
  3. `cargo test --workspace --tests --lib` — must pass. (Skip doctests
     locally if your toolchain lacks `rustdoc`; CI runs them.)

  These three commands take under a minute on a warm target dir.
  Running them before pushing prevents the "push → wait 10 min → red CI
  → push fixup" loop. The agent is responsible for this gate; do not
  rely on the human to catch it.

- **Size-budget gate (run before EVERY `git push`, and before every tag
  / release):** `./tests/check.sh --size-budget`. This builds the
  canonical ship artefact (`release-slim`, glibc `cpu`, default
  features) and asserts the **exact** thing CI's `size-budget` job
  asserts — binary ≤ the `cpu` budget (currently 25 MiB / 26,214,400 B)
  and a four-entry `NEEDED` allowlist (libc, libm, libgcc_s, the dynamic
  linker). The numbers live in lockstep with the `ci.yml` `cpu` matrix
  rows; change them together. A green run here means the CI size gate
  will pass — so binary growth never surprises us at CI time. The flag
  runs **only** the size gate (it skips the fmt/build/clippy/test matrix
  and does its own dedicated build), so it composes with the three
  commands above. It pins `libonnxruntime.a` via `ORT_LIB_LOCATION`
  exactly as CI (auto-resolving through `scripts/fetch-onnxruntime.sh`
  when the env var is unset). If the binary is over budget, fix the
  growth or, with sign-off, bump the `cpu` row in both `ci.yml` and
  ADR 0022 (hard cap ≤ 28 MiB) — never silently exceed it.

  Style note: `rustfmt.toml` sets `use_small_heuristics = "Max"` so
  short fn calls / struct literals / if-else expressions stay on one
  line when they fit in `max_width = 100`. Compact code is preferred.
  For the rare case rustfmt insists on expanding a genuinely tasteful
  one-liner (e.g. `fn ok(s: &str) -> String { paint("32", s) }`),
  prefix the item with `#[rustfmt::skip]` rather than fighting the
  formatter codebase-wide.

- All commits **MUST** be signed off (`git commit -s`) — DCO enforced by CI.
- **NEVER** add a `Co-authored-by: Forge <forge@noreply.local>` trailer (or any
  agent / assistant co-author trailer) to commit messages — not on new commits,
  not when rewording, not when squashing. The agent is a tool, not an author.
  When squashing history, strip any pre-existing `Co-authored-by: Forge …`
  lines from the combined message. This rule is permanent.
- **Commit messages MUST be user-friendly.** Write the subject and body so a
  non-expert user reading the changelog / release notes / `git log` can easily
  understand *what changed for them and why* — describe the behaviour or
  benefit in plain language, not the internal mechanics. Prefer e.g.
  "Make the wake word trigger more reliably and stop false activations" over
  "Replace score smoother with sliding-window activation gate". Keep jargon,
  type/function names, and implementation detail out of the subject line; if

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bogdanr/fono](https://github.com/bogdanr/fono) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-06 -->

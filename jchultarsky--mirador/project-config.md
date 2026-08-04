---
trigger: always_on
description: Context for anyone (human or agent) picking this repo up cold. Design rationale
---

# CLAUDE.md — working notes for mirador

Context for anyone (human or agent) picking this repo up cold. Design rationale
that belongs to users lives in `README.md` and `CONTRIBUTING.md`; this file is
the stuff you would otherwise have to reverse-engineer from the diff.

There was a `HANDOFF.md` alongside this carrying the perishable half — where
work stopped, what was waiting on a decision. Its open items are all either
done or folded in below, so it has been deleted as it asked to be. Write
another the same way if you hand off mid-flight, and delete it the same way.

## What this is

A personal information dashboard for the terminal, in Rust + ratatui. Clock,
weather, tasks, and live system metrics in a config-driven grid. MIT licensed.
Repo: `github.com/jchultarsky/mirador`. Owner: Julian Chultarsky.

A *mirador* is a lookout — the tower you climb to see everything at once.

**Known name collision, accepted knowingly:** Project Mirador, the IIIF image
viewer. Raised before the name was chosen; not a reason to revisit it.

## The one-line pitch that shapes every decision

A tab you leave open all day and come back to. That constrains more than it
sounds like it does: no ambient blinking, no shimmering graphs, no doomscroll
hooks, calm by default so that *not* calm is legible at a glance.

## Commands

```sh
cargo test                                    # all fast, no network, no fixtures
cargo clippy --all-targets -- -D warnings     # must be silent
cargo fmt --all -- --check                    # must be silent
RUSTDOCFLAGS="-D warnings" \
  cargo doc --no-deps --document-private-items  # CI runs this; it caught a
                                                # broken intra-doc link that
                                                # the other three did not
cargo run -- --print-config > /tmp/m.toml     # scratch config to experiment on
cargo run -- --config /tmp/m.toml
```

**The bar is zero warnings and zero errors across all four.** Running only the
first three is how a red build reached `main`: rustdoc rejects a link to a
`cfg(test)` item, and nothing else notices.

`rust-version` is driven by dependencies, and `cargo` reports only the *first*
blocker — chasing them one at a time costs a CI round trip each. Get the real
floor in one go:

```sh
cargo metadata --format-version 1 | \
  jq -r '[.packages[].rust_version | select(.)] | max'
```

It currently comes from `sysinfo`, not from anything this crate does. Five
places have to agree: `Cargo.toml`, `.github/workflows/ci.yml` (twice), the
README and CONTRIBUTING.

Note that `rust-version` also changes what clippy suggests — raising
the MSRV to 1.88 turned every nested `if let` into a `collapsible_if` error,
because let-chains only became available there, and 1.88 → 1.95 turned every
`Duration::from_secs(3600)` into one, because `from_hours` had stabilised.
Expect a round of mechanical fixes with any bump, and do not take the
suggestion on faith: clippy will offer `from_days`, which is *still unstable*
at the 1.95 floor and would break the very job the bump was fixing.

This cuts the other way too, and it is worth knowing before you go hunting.
**Clippy gating means a local run and CI can legitimately disagree**: bump
`rust-version` locally, run clippy, and you will see lints a CI still on the
old floor does not. That is the gate working, not CI failing. Compare the
`rust-version` each side is using before concluding anything is broken.

The crate enables `clippy::pedantic`. When a lint is genuinely wrong, add a
targeted `#[allow]` *with a comment saying why* — do not widen the allow list
in `Cargo.toml`.

**`assets/default_config.toml` is `include_str!`-baked into the binary.**
Editing it does nothing until you rebuild. This has twice looked like a change
that did not land when it simply had not been compiled.

To eyeball the rendering without a terminal:

```sh
cargo test dump_dashboard -- --ignored --nocapture   # renders to stdout
```

To drive it in a real terminal headlessly, run it under `tmux` and
`tmux capture-pane -p`. Several layout bugs only showed up this way — the
`TestBackend` dump will not catch a panel whose content is pushed off the
bottom, because nothing errors. `-e` keeps the escape sequences, which is the
only way to prove a colour bug; a capture of black-on-black shows nothing
either way.

**A test that cannot fail is documentation with a `#[test]` on it.** The id
reuse bug shipped with `ids_are_unique_and_survive_deletion` sitting directly
on top of it: the test compared the new id against the *surviving* task rather
than the *removed* one, so it passed throughout. When you fix a bug that had a
test nearby, check that the test would have caught it — break the fix on
purpose and watch it go red. Twice now that check was the difference between a
real test and a reassuring one.

## Architecture

```
main.rs      CLI parsing, terminal setup
app.rs       event loop, focus ring, grid geometry, help overlay, status bar
panel.rs     the Panel trait — the seam every widget goes through (in-tree)
frame.rs     panel frames, Binding type, key hints punched into borders
grid.rs      shared column grid with named headers
chart.rs     braille graphs + baked colour gradients

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jchultarsky/mirador](https://github.com/jchultarsky/mirador) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->

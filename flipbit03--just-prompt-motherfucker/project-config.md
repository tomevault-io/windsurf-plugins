---
trigger: always_on
description: A manifesto site. One Rust binary serves a page compiled from
---

# CLAUDE.md

A manifesto site. One Rust binary serves a page compiled from
`manifesto/MANIFESTO.md` and records GitHub-authenticated signatures in SQLite.

## Layout

```
manifesto/MANIFESTO.md   the text; the binary embeds it with include_str!
app/                     the whole website — one crate, ~2,000 lines
  src/main.rs            config, routes, handlers
  src/render.rs          markdown -> HTML, the page, the head metadata
  src/db.rs              schema and queries
  src/github.rs          OAuth against GitHub
  src/cookies.rs         the two CSRF defences
  assets/style.css       inlined into <head> at build time
infra/rustible/          provisions the host (Rustible, not Ansible)
infra/jpmf.service       systemd unit, shipped by the deploy
.github/workflows/       ci.yml, deploy.yml
```

## Stack

axum + tokio, rusqlite (`bundled`, so SQLite compiles in), pulldown-cmark,
reqwest (rustls), clap, sha2, getrandom. Rust edition 2024, pinned by
`rust-toolchain.toml`. Ships as a static musl binary, ~8 MB.

## Working on it

```sh
cd app
cargo run                                  # http://localhost:8100
cargo fmt --check && cargo clippy --all-targets -- -D warnings && cargo test
```

CI runs exactly those three. Clippy is `-D warnings`; treat a warning as a
failure locally too.

Changing `manifesto/MANIFESTO.md` requires a rebuild — `include_str!` makes it a
build dependency, so `cargo run` picks it up automatically.

### Looking at the page

If `agent-browser` is on the machine, use it after a layout or stylesheet change
instead of assuming the change landed.

```sh
agent-browser --session jpmf set viewport 1280 900
agent-browser --session jpmf open http://localhost:8100/
agent-browser --session jpmf screenshot /tmp/shot.png
agent-browser --session jpmf close
```

`set viewport 390 844` for the phone layout, `set media dark` for the dark
palette. Both have caught things that looked fine in the markup.

Measure rather than eyeball when the question is spacing or alignment — `eval`
returns element rectangles:

```sh
agent-browser --session jpmf eval 'JSON.stringify(document.querySelector("hr").getBoundingClientRect())'
```

That is how the gap above and below the `****` divider was found to be uneven,
and how it was confirmed even afterwards.

## How a request works

```
GET  /                  manifesto + signature list, rendered from SQLite per request
POST /sign              CSRF check -> set state cookie -> 303 to GitHub
POST /unsign            same, with the intent encoded in the state
GET  /auth/callback     verify state -> exchange code -> read account -> act
POST /unsign/confirm    consume a pending token -> delete the row
GET  /healthz           the deploy gates on this
GET  /robots.txt
```

Anything else answers a rendered 404.

### Signing is not a login

There is no session. The OAuth token is used once to read the account id and
dropped; nothing about a visitor is remembered between requests. `scope` is
empty, so GitHub grants only public profile data.

Removing a signature runs the same flow again — re-authenticating *is* how
someone proves the row is theirs. The `pending_unsign` row carries the proven
identity from the callback to the confirm button for five minutes, single use,
pinned to that signature's `signed_at` so a stale confirmation cannot delete a
signature made after it was issued.

Two separate CSRF defences, covering different holes:

- **`jpmf_state`** covers the callback. Compared whole against the query
  parameter; a forged query string cannot match a cookie the attacker could
  never set.
- **`jpmf_csrf`** plus a hidden form field covers the way *in*. Without it a
  cross-site POST could start a flow on a visitor's behalf — GitHub skips the
  consent screen for anyone who already authorised the app.

## Data

One table, `signatures`, defined in `app/src/db.rs`. Keyed by `github_id`,
because handles get renamed and ids do not.

WAL, `synchronous=NORMAL`. One connection behind a `Mutex`; never hold the lock
across an `.await`. Schema is `CREATE TABLE IF NOT EXISTS` at startup — no
migration framework.

## Invariants

- **`manifesto/MANIFESTO.md` is the only copy of the text, and its opening
  lines are load-bearing.** `render::front_matter` parses the file at startup:
  the `# ` line becomes `<title>`, the `# ` line and the emphasised line under
  it together become the share card's title and alt text, and the first
  paragraph of prose becomes the description search engines and share cards
  show. Change the *shape* of those lines — drop the emphasis, add a paragraph
  above the title, lead with a quote — and the page's metadata changes with it
  or empties out. The manifesto carries a comment
  saying so. These were Rust constants once; they drifted, and the share card
  advertised a subtitle the document no longer had.
- **The numbers beside names are positions**, computed at render time from
  `signed_at` order. Nothing stores a display number. Removing a signature
  closes the gap; the founders occupy the first positions so the list starts
  after them.
- **Founders are a const, not rows.** `db::FOUNDERS`. Nothing to insert and
  nothing to delete, so they cannot be revoked and both flows refuse early.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [flipbit03/just-prompt-motherfucker](https://github.com/flipbit03/just-prompt-motherfucker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-18 -->

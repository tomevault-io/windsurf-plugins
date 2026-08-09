---
trigger: always_on
description: Notes for whoever (human or agent) works on this codebase next.
---

# AGENTS.md

Notes for whoever (human or agent) works on this codebase next.

## Layout

```
src/
├── main.rs            # clap CLI: import / list / open (default) / info / edit / rename / delete / path
├── lib.rs             # re-exports the modules below for the binary + tests
├── app.rs             # App state, vim mode machine, ratatui run loop
├── input.rs           # keymap: Normal / Insert / Command modes, popups
├── ui.rs               # rendering: sidebar, url bar, editor tabs, response, popups
├── highlight.rs         # JSON/XML tokenizer -> styled ratatui Lines
├── model.rs            # Collection / SavedRequest / KeyValueRow / FieldDoc / OAuthConfig+AuthKind (serde)
├── store.rs            # ~/.config/cielago persistence, AppConfig
├── openapi/
│   ├── loader.rs        # load spec from file path or http(s) URL, JSON/YAML
│   ├── resolve.rs        # local `#/...` $ref resolution (cycle-safe)
│   ├── examples.rs        # schema -> example JSON value generation
│   ├── docs.rs             # schema -> FieldDoc (types, enums) for the Docs tab
│   └── import.rs            # Spec -> Collection conversion
└── http/
    ├── client.rs           # reqwest request building + response capture
    ├── oauth.rs              # client-credentials token exchange
    ├── secret.rs              # $(cmd) secret resolution via `sh -c`
    ├── send.rs                # send_with_auth: pick scheme; oauth cache/refresh + 401 retry
    ├── url_input.rs            # pasted URL -> origin / path / query (inverse of build_url)
    └── vars.rs                 # {{name}} + dynamic ({{uuid}}, {{randomInt}}…) substitution
```

`tests/` has fixture-driven integration tests: `import_tests.rs` (spec →
collection), `http_tests.rs`/`app_send_tests.rs` (wiremock-backed HTTP +
OAuth), `input_tests.rs` (full keymap flows over an in-memory `App`),
`ui_tests.rs` (draws into a ratatui `TestBackend` and asserts on cell colours
— the only place rendering is covered).

## Design decisions worth knowing

- **The project was renamed `getman` → `stableman` → `manpost` → `cielago`.**
  `store::config_dir` moves a leftover `~/.config/{manpost,stableman,getman}`
  onto `~/.config/cielago` on first use (see `LEGACY_DIR_NAMES`), so existing
  collections survive. Drop those migrations once they've had time to run
  everywhere.
- **No remote `$ref`s.** `openapi::resolve` only follows local
  `#/components/...` JSON pointers. Specs that split across files aren't
  supported — bundle them first if you hit this.
- **Auth is one struct, three schemes.** `OAuthConfig` (aliased `AuthConfig`)
  carries every scheme's fields, discriminated by `AuthKind` (`bearer` /
  `apikey` / `oauth2`). It stays a single flat JSON object so collections
  written before bearer/api-key support — whose `auth` has no `kind` — still
  deserialize (missing `kind` defaults to `oauth2`, which is what they were).
  The popup (`A`) builds its rows from `App::auth_fields` per kind; the first
  row is always the kind toggle. `send::send_with_auth` branches on the kind:
  bearer/apikey resolve their secret and send (no token cache, no 401 retry),
  oauth2 keeps the cache-and-retry path.
- **Secrets are plaintext, but can be indirected.** Secret fields
  (`token`, `client_secret`) are saved as-is in the collection JSON (explicit
  user choice). To avoid that, a field may hold a single `$(…)` command
  substitution — `http::resolve_secret` runs it through `sh -c` at send time
  and uses the trimmed stdout. Only a value that is *entirely* `$(…)` is
  executed, never one embedded in a longer string. The in-memory `OAuthToken`
  is never persisted.
- **Tags become sidebar groups**, first tag only; untagged requests land in
  a `default` group. This wasn't asked for explicitly but was cheap and
  matches how most specs are organized.
- **`Method::parse`, not `FromStr`** — deliberately not the trait, to dodge
  a clippy lint; nothing else depends on `FromStr`.
- **Vim modes are `Normal` / `Insert` / `Command` / `Search`** — no Visual
  mode. Insert mode is only single-line field edits (`LineEdit`), tracked by
  `app.editing`; there is no in-app multi-line editor. `Search` is the
  `/` sidebar filter: it re-applies on every keystroke, and `app.filter` (the
  committed query) is deliberately separate from `app.search` (the live
  prompt buffer) so `Esc` can drop the prompt without touching the filter.
- **Sidebar labels are a view concern.** `SavedRequest` keeps `name` (user
  editable), `summary` and `operation_id` (verbatim from the spec);
  `Collection.label_mode` picks which one renders. `:rename-all` is the only
  thing that overwrites `name`. Import prefers `summary` over `operationId`
  for the initial name — most real specs put a generated controller method
  name in `operationId`.
- **Switching collections reassigns the whole `App`** (`switch_collection` does
  `*self = App::new(...)`). Everything view-related is derived from the
  collection, so there's nothing to migrate by hand — and dropping the mpsc
  channel and cached `OAuthToken` is a feature, not collateral: a response still
  in flight for the previous collection can no longer land in the new one, and
  the token belonged to the old `auth` config. It deliberately does *not* go

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stevedylandev/cielago](https://github.com/stevedylandev/cielago) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->

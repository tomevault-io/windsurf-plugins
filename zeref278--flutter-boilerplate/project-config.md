---
trigger: always_on
description: Conventions and invariants for anyone — human or agent — changing this code.
---

# Working in this repository

Conventions and invariants for anyone — human or agent — changing this code.
The README explains what the project *is*; this file is what you need to not
break it.

## Commands

```sh
make setup                              # deps + all code generation
make analyze                            # dart format --set-exit-if-changed + dart analyze
make test                               # unit and widget tests
make run_dev                            # bake .env.dev, then run
make build_apk_production               # bake .env.production, then build
make coverage COVERAGE_DEVICE=<ios-id>  # merged trace, 80% gate
make rename NAME= ORG= DISPLAY=         # rename the project
make flavorize                          # regenerate native flavor scaffolding
```

FVM is optional; targets use `fvm` when installed and plain `flutter` when
not. Override with `make test FLUTTER=... DART=...`.

Before saying work is done: `make analyze` and `make test` both clean.

## `dart analyze` must report nothing

Zero issues — not zero errors, zero *issues*. That is the contract. If your
change introduces one, fix it rather than raising a baseline, and do not
reach for `// ignore:`.

Two shapes recur, both already solved in the tree:

- **`prefer_initializing_formals`.** Dart has no private *named* initializing
  formal, so `({required X x}) : _x = x` cannot be written the way the lint
  asks. Use a positional `this._x`, or make the field public `final`.
- **`avoid_catching_errors`.** When a library signals a recoverable condition
  with an `Error` subtype, match it inside the block and rethrow the rest —
  see `_openOrRecreate` in `encrypted_store.dart` — rather than naming it in
  the `on` clause.

## Invariants

These are load-bearing. Breaking one is a bug even if it compiles.

### `core` names no feature

`grep -ri dog_image lib/core` must return nothing. Widget keys live with the
surface, feature tables live with the feature, and `Injector` is handed its
module list rather than importing one.

The two files allowed to know every feature are flat lists outside `core`:

| File | Names every | Adding a feature |
|---|---|---|
| `lib/di/app_modules.dart` | DI module | one line |
| `lib/database/app_database.dart` | Drift table | one entry, plus the table under the feature |

### Features import `core` through one barrel

`package:boilerplate/core/core.dart`. It is a **curated** export list, not
`export everything`. Storage backends (`EncryptedStore`, `Keychain`,
`AppStorageImpl`) and composition wiring (`CoreModule`, `createDio`, the log
and crash services) are absent on purpose — that is what keeps "application
code injects only `AppStorage`" enforced by imports rather than by convention.

Nothing inside `core` may import the barrel, or a cycle forms. Tests import by
path, because they deliberately reach the internals the barrel omits.

### Configuration is compiled in, never read at runtime

Values come from one `.env` file baked by `envied` at code-generation time.
`AppConfig` is the facade; **nothing outside `lib/config/env/` reads `Env`**,
and nothing reads a `.env` file at runtime.

Never edit `lib/config/env/env.g.dart`. To change a value, edit the `.env`
file and re-run `make env_<flavor>`. Editing a `.env` in place and rebuilding
the same flavor can reuse a cached result — run `make clean_gen` if a value
looks stale.

Adding a flavor means adding it in **five** places, or it half-exists:
`.env.<flavor>`, `flavorizr.yaml`, `AppFlavor`, `FLAVORS` in the `Makefile`,
and `ios/Runner/Assets.xcassets/AppIcon-<flavor>.appiconset`.

### Security vendor is named in two files only

`lib/core/security/rasp_secure_app_guard.dart` and
`rasp_network_security.dart`. Everything else sees `SecureAppGuard`,
`SecureAppThreat`, and `NetworkSecurity`. Same principle keeps Hive inside
`EncryptedStore`.

`security` is a **required** argument on `createDio`. Keep it required, so a
new caller is a decision about transport security rather than an omission.

### Never log a credential

A credential reaches a log through three doors, and all three are shut:

- **Headers** — redacted by name against `defaultRedactedHeaders`. Add any
  new secret-carrying header to that deny-list.
- **Query strings** — redacted by name against `defaultRedactedParams`.
  `?api_key=` reads as plainly in scrollback as any header.
- **Bodies** — not written unless `logBodies: true`. A login body's password
  and a token endpoint's response are the payload, under a key this class
  cannot guess, so the default is to write nothing. Turning it on is a
  deliberate debugging choice, not something to leave on.

Do not reintroduce `pretty_dio_logger` — it prints `options.headers`
verbatim, which is why it was removed.

### Transient messages go in the overlay, never on the Navigator

`AppToast.show` inserts an `OverlayEntry`. Do not reach for
`another_flushbar` — it was removed because it shows its bar by *pushing a
route*, so the back button dismissed it, `RouteObserver`s counted it as a
navigation, and a `pop` during its lifetime popped the bar instead of the
page. A notification is not a destination.

`ScaffoldMessenger` is not the answer either — but not for the reason it is

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zeref278/flutter_boilerplate](https://github.com/zeref278/flutter_boilerplate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->

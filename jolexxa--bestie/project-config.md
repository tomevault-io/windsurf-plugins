---
trigger: always_on
description: We value contributions from our robot friends — we just ask that they carefully and thoughtfully adhere to the best practices listed below.
---

# AGENTS

We value contributions from our robot friends — we just ask that they carefully and thoughtfully adhere to the best practices listed below.

## Code Quality

We prefer code that embodies clear, concise mental models. We prefer to think deeply about the problem we are solving and find the solution that best fits.

- Example 1: a file with many boolean variables might be implemented more cleanly as a state machine (using a bloc, cubit, or other package/pattern).
- Example 2: a file with a series of complex async operations may be better described as a series of stream transforms, an observable primitive, or even a composite.

If you recognize a key insight that would clean something up but do not have what you need on hand to implement it, please just say so. Adding a package reference is easy.

Our criteria for good code also enables us to achieve 100% test coverage.

Good code has...

- as few branches as possible
- injectable dependencies
- well-named identifiers
- no sibling dependencies in the same architectural layer

To avoid sibling dependencies, state must either be lifted up to a common ancestor and passed down, or pushed down and subscribed to.

See README.md for full development setup and contributing guidelines.

We are in the process of exterminating all record types in the codebase. If you work on a file with record types, please help me identify and remove them.

## Testing

When writing tests, write tests that assert the code does what it *should* do, not what it actually does.

We employ mocktail for mocking dependencies of systems under test. We only use custom fake objects when we cannot mock (such as FFI boundaries). We do not spin up real dependencies in tests -- only mocks or fakes.

We use the right tool for the job when testing: mocking (mocktail), clock (time), fakeAsync, platform, etc.

## Developer Scripts

Melos owns workspace orchestration. Run commands from the repo root.

```bash
dart pub get                                   # install workspace dependencies
dart run melos run test --no-select           # run Dart tests
dart run melos run analyze --no-select        # dart analyze --fatal-infos
dart run melos run format --no-select         # format root tools and packages
dart run melos run format:check --no-select   # CI formatting check
dart run melos run coverage --no-select       # tests + lcov coverage report
dart run melos run codegen --no-select        # build_runner code generation
dart run melos run ffigen --no-select         # standard ffigen packages
dart run melos run ffigen_posix_macos         # POSIX macOS bindings
dart run melos run ffigen_posix_linux         # POSIX Linux bindings
dart run melos run build:spawner              # build the spawner PTY helper (Rust, POSIX only)
dart run melos run build:sidecars             # build every Rust sidecar (brush, coreutils, ripgrep, findutils, sed, bestie_edit)
dart run melos run build:edit                 # build just bestie_edit (also build:brush, build:coreutils, build:ripgrep, build:findutils, build:sed)
dart run melos run checks:edit                # cargo fmt / clippy / test for the bestie_edit crate
dart run melos run checks:guard               # cargo fmt / clippy / test for the bestie_guard sandbox crate
dart run melos run setup                      # full fresh-clone bootstrap (host-aware; see below)
dart run melos run checks --no-select         # full CI check sequence
dart run melos run credits                    # regenerate CREDITS.md from shipped dependencies
```

## Tool Calls

Tool calls span across the architecture layers to support extensibility and composition of the app at the highest level. A tool call that goes to the background (i.e., becomes a job) owns its output until the conversation ends.

## Native Binaries

See [APP_ASSETS.md](APP_ASSETS.md) for how shipped files work end to end: the app-asset manifest, the native-asset build hooks, where generated vs vendored assets live, and how a release bundle is assembled. Read it before adding or moving one.

Native libraries for FFI packages are not checked into git — the one exception is vendored files under the repo-root `assets/`, which is the only place a shipped binary may be committed. After cloning, run:

```bash
dart tool/download_curl_assets.dart         # curl-impersonate native libs
dart tool/download_openconsole_assets.dart  # Windows console host (conpty.dll + OpenConsole.exe)
dart run melos run build:spawner            # `spawner` PTY helper (Rust → packages/ffi/posix_spawner/assets/native/<os>/<arch>/spawner)
dart run melos run build:sidecars           # brush, coreutils, ripgrep, findutils, sed (Rust → packages/infra/agent_shell/assets/native/<os>/<arch>/shell/bin)
                                            # and bestie_edit (Rust → packages/infra/bestie_edit/assets/native/<os>/<arch>/bestie_edit)
```

Or just `dart run melos run setup` (→ `tool/setup.dart`) for the whole fresh-clone sequence: submodules, deps, native assets, sidecars (spawner plus everything `build:sidecars` covers), codegen. It is host-platform-aware.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jolexxa/bestie](https://github.com/jolexxa/bestie) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-10 -->

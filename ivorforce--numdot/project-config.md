---
trigger: always_on
description: NumDot is a numpy-like tensor math library for the Godot game engine.
---

# NumDot

NumDot is a numpy-like tensor math library for the Godot game engine.
NumDot imitates numpy's API where possible, using the `nd` class with static methods like `nd.add(a, b)`.

Find its documentation at `./docs/`. The source code is in `src/`.

## Building

To build for the `./demo` godot project:

```
scons install_dir=demo api_version=4.3
scons install_dir=demo arch=arm64 api_version=4.3  # on macOS
```

## When making changes to NumDot's behavior

Edit `./docs/setup/changelog.rst`. If the "Upcoming Changes" is commented out, uncomment it. Be brief. Match the style of existing changelogs.

The changelog is always in reference to the previous released version. When making changes on `main` to unreleased behavior, edit the changelog instead of adding stacking entries.

The changelog is user-facing. It should document what the user sees or what it might mean to them, not internal implementation details. Architectural changes are only briefly mentioned, and focused on the implication.

## When changing the public C++/GDScript API

After building, regenerate the docs:

```bash
cd demo && /Applications/Godot-4.4.app/Contents/MacOS/Godot --doctool ../ --gdextension-docs --headless && cd ..
tests/.venv/bin/python configure/regenerate_docs.py
```

Then fill in any new descriptions in `doc_classes/*.xml` and re-run the wrapper. The output is idempotent across machines and ready to commit.

Gotcha: `doctool` reads the **currently-built** binary. Methods missing from your build (e.g. an incomplete or stale `.so`) silently disappear from the XML. Verify the diff before committing.

## Testing & iteration

What follows is for working on the test infrastructure. Build setup and
project-wide conventions live in `Readme.md` / `CONTRIBUTING.md`.

### The two test surfaces

- **Legacy smoke**: `tests/run_tests.py` — generator-based; spawns Godot
  once, dumps `.npy` files, diffs against numpy. Works on Godot 4.3+.
- **TCP bridge stack**: `tests/bridge/` (transport + sanity tests),
  `tests/numdot_xp/` (Array API adapter), `tests/array-api-tests/`
  (upstream conformance suite, git submodule), `tests/array_api/`
  (skips/xfails/README). Requires Godot **4.4+**.

`tests/` is a Poetry project with an in-project `.venv`; run
`(cd tests && poetry install)` after pulling.

### Run commands

Both suites use `tests/.venv/bin/python` directly (poetry-managed). Set
the Godot path to a 4.4+ binary; many versions exist on this machine
under `/Applications/Godot-*.app/Contents/MacOS/Godot`.

Bridge sanity (fast, ~25s):

```bash
tests/.venv/bin/python -m pytest tests/bridge/ \
  --godot=/Applications/Godot-4.4.app/Contents/MacOS/Godot
```

Array-API steady state (must stay green; new failures = regressions):

```bash
PYTHONPATH=tests \
NUMDOT_GODOT=/Applications/Godot-4.4.app/Contents/MacOS/Godot \
ARRAY_API_TESTS_MODULE=numdot_xp \
ARRAY_API_TESTS_XFAIL_MARK=skip \
  tests/.venv/bin/python -m pytest tests/array-api-tests/array_api_tests/ \
    --skips-file tests/array_api/skips.txt \
    --xfails-file tests/array_api/xfails.txt \
    --hypothesis-disable-deadline
```

Triage mode (lets `XPASSED` show up so you can de-xfail tests that have
started passing): drop the `ARRAY_API_TESTS_XFAIL_MARK=skip` line.
Iteration: add `--max-examples 3 -q --tb=no` for sub-second cycles on
one file.

### Iteration loop for conformance work

Goal: drive `xfails.txt` to zero and bring more files out of `skips.txt`.

1. Pick a target — either a file in `skips.txt` to enable, or a section
   in `xfails.txt` to clear.
2. Run that one file with low `--max-examples` for fast feedback.
3. For each failure, decide:
   - **Missing/wrong adapter shim** → edit `tests/numdot_xp/_funcs.py`.
   - **NumDot signature mismatch** → adjust the shim (NumDot is
     positional-only; Array API is keyword-heavy).
   - **NumDot bug or missing function** → file an issue, leave xfailed.
4. Move the line out of `xfails.txt` (or add it with a section header
   when introducing a new failure).
5. Re-run the steady-state command above; commit.

Always re-run the bridge sanity suite after touching `tests/bridge/` or
`demo/tests/bridge.gd`.

### Architecture facts that are easy to forget

- `nd` in GDScript is a **class with only static methods** — not an
  instance. `nd.has_method(...)` and `nd.callv(...)` are *parse errors*.
  Use `ClassDB.class_has_method("nd", name)` and
  `Callable(ClassDB, "class_call_static").callv(["nd", name, ...args])`.
  `class_call_static` is **Godot 4.4+ only**.
- GDScript's `JSON.parse_string` is not bit-exact for numbers — it
  parses every number as `float64`, losing int precision above 2^53
  *and* off-by-one-ULP on some float edge values. The wire encoder
  routes ints as strings and floats/numpy scalars as 0-d `.npy`
  blobs so every numeric scalar arg survives bit-exact.
- The bridge runs Godot in a child process. **NumDot segfaults kill
  Godot, not pytest.** `BridgeClient` detects the dead pipe, marks
  itself dead, and `numdot_xp` respawns transparently before the next
  test. No need for `pytest-xdist` for isolation.
- The bridge lives entirely in `demo/tests/bridge.gd` (pure GDScript).
  No bridge code is in the GDExtension — zero risk of leaking into
  shipped binaries.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ivorforce/NumDot](https://github.com/Ivorforce/NumDot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->

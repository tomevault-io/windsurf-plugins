---
trigger: always_on
description: Operating rules for AI agents working on **Afterglow**. These are not suggestions.
---

# AGENTS.md

Operating rules for AI agents working on **Afterglow**. These are not suggestions.
They describe patterns that already exist in this codebase and that every change must
preserve. When in doubt, read the module you are about to touch and the format document
it cites in `docs/`.

Afterglow builds and flashes configuration files for Harmony remote controls whose
vendor service (Logitech) has shut down. A wrong config on a remote nobody has tested
can brick hardware that **cannot be re-flashed from a vendor server any more**. Treat
the format and the safety gates as load-bearing, because they are.

---

## 1. Tech stack - do not add to it

| Layer | What | Constraint |
|---|---|---|
| Language | Python `>=3.11` | Uses `X \| None`, `from __future__ import annotations`. Keep it. |
| Format / build layer | Pure standard library | **Zero third-party imports.** `pyproject.toml` `dependencies = []` is deliberate: unpacking, rebuilding and verifying a config must work on a bare Python. |
| GUI | `PyQt6>=6.5`, `Pillow>=10` | Optional extra `[gui]`. GUI code lives **only** under `src/afterglow/gui/`. |
| Tests | `pytest>=7` | Optional extra `[dev]`. |
| Lint | `ruff`, `line-length = 100` | Config lives in `pyproject.toml`. Not installed by default; respect the 100-col limit anyway. |

**Rules**

- Do **not** add a runtime dependency to the format/build layer. If you think you need
  one, you are solving the problem in the wrong layer.
- PyQt6 and Pillow may be imported **only** from `src/afterglow/gui/`. Nothing outside
  `gui/` may `import PyQt6` or `from PIL`. The build layer is headless so the CLI,
  the tools and the tests run without Qt or a display.
- Target 3.11. Do not use 3.12+-only syntax.

---

## 2. File structure - where things go

> ### The rule this section exists to serve
>
> **Every piece of code is remote-agnostic unless it lives inside that remote's
> `backends/<name>/` folder or a `payloads/<type>.py` module.** Removing support for a
> remote or a container format must be `rm -r` - never a refactor. No tight coupling in
> either direction.
>
> `tests/test_removability.py` enforces this by *doing* it: it builds the package with
> one backend, then one payload type, deleted, and imports everything that is left. It
> also fails any shared module that imports a concrete component or resolves one by a
> hardcoded name.
>
> Violations of this rule do not announce themselves. They look like: a static import of
> a concrete component in shared code, a `DEFAULT` that names one, a check for a path only
> one format has, a profile field defaulting to a component name, or a lookup by
> hardcoded id. Each one turns `rm -r` into a refactor, and one of them silently opens an
> unidentifiable config as a Harmony 900.
>
> The shapes that replaced them are the patterns to copy: **components are discovered,
> not listed**, and **they declare what they own** rather than shared code declaring it
> for them - `sniff()`/`claims()` for payload types, `claims_legacy()` for backends, and
> `LAST_RESORT` for a type that accepts anything so it can never win by default.

```
afterglow.py              launcher; puts src/ on the path FIRST, then imports the package
pyproject.toml            src-layout, package data (library/icons/scaffolds), ruff, pytest
src/afterglow/
  paths.py                the ONE answer to "where does shipped data live" - see §4
  ezhex.py                the .ezhex container: unpack / pack_standalone / rehash
  project_devices.py      canonical portable device shape stored in projects
  remotes.py              remote profiles + the writable/untested safety gate
  importer.py             profile-selected backend import dispatcher
  library.py              private portable device/protocol library + learn()
  build_service.py        headless build orchestration (ConfigBuildService)
  vocabulary.py           device types, activity types and hard keys - read from the
                          remote's own profile, not held here (button glyphs are
                          files under icons/buttons/, found via paths.icons())
  remote_shell.py         a shell on the remote over USB: its own system log
  ir_signal.py            PORTABLE signal envelope - no backend may be imported here
  ir_protocol.py          PORTABLE protocol grammar - likewise
  device_json.py          portable database record -> portable project device
  rf.py, hao.py, concord.py, flipper.py ...
  backends/               ONE ARCHITECTURE'S ENCODING. Selected by the remote profile.
    __init__.py           dynamic registry + required backend contract
    harmony_pk/           PK\x03\x04 payload import, lowering and build: builder/, importer,
                          states, irproto, ssir, protocol_json, ir_emit, ir_vm and the
                          generated-native registry
  payloads/               container formats, selected by profile.payload: pk, blob
  gui/                    PyQt6 authoring app ONLY
    app.py                MainWindow + main() entry point
    tabs.py               the four tabs: Devices / Activities / Remote Settings / Flash
    device_wizard.py activity_wizard.py macro.py properties_editor.py ...

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [harmonic-afterglow/afterglow](https://github.com/harmonic-afterglow/afterglow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-05 -->

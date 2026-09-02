---
trigger: always_on
description: Guidance for Claude Code and developers working in this repository.
---

# CLAUDE.md — Developer Guide

Guidance for Claude Code and developers working in this repository.

## Overview

Android Emulator Skill provides scripts for Android app building, testing, and automation, wrapping
`adb`, `uiautomator`, and the Gradle wrapper with semantic, token-efficient interfaces designed for
AI agents. It is the Android counterpart of `ios-simulator-skill` (credited in the README).

## Project structure

```
android-emulator-skill/                      # repo root
├── .claude-plugin/marketplace.json          # marketplace manifest (owner: fluxxion82)
├── android-emulator-skill/                   # distributable plugin package
│   ├── .claude-plugin/plugin.json
│   └── skills/android-emulator-skill/
│       ├── SKILL.md                         # entry point / script reference + roadmap
│       └── scripts/                         # production scripts + common/ utilities
├── .github/workflows/                        # lint, test, release, validate-version, pages
├── tests/                                    # pytest suite (adb/subprocess mocked)
└── pyproject.toml / .pre-commit-config.yaml
```

## Architecture patterns

- **Class-based scripts** with a thin `main()` that parses args and prints results.
- **Serial resolution**: scripts accept optional `--serial` and auto-detect via
  `common.device_utils.resolve_device_identifier`. (Note: `get_device_serial` does **not** exist —
  use `resolve_device_identifier`.)
- **Output modes**: concise default (3–5 lines) → `--verbose` → `--json`.
- **Batch operations** where sensible (`--all`).

## Shared utilities (`scripts/common/`)

- **device_utils.py** — `build_adb_command()` (never `shell=True`), device detection, and
  `get_ui_hierarchy()`.
  - ⚠️ **Hierarchy contract**: `get_ui_hierarchy()` returns nodes shaped as
    `{"tag": str, "attributes": {<raw XML string attrs>}, "children": [...]}`. All UI fields
    (`class`, `text`, `bounds`, `clickable`, `content-desc`, `resource-id`, …) live **under
    `node["attributes"]`** as **strings**. Consumers must read from `attributes` and parse types
    themselves (e.g. `bounds` is `"[l,t][r,b]"`; booleans are `"true"`/`"false"`). Reading fields
    directly off the node (`node.get("class")`) silently returns nothing — this was a real bug.
- **screenshot_utils.py** — capture/resize; **inline mode returns the image under the
  `base64_data` key** (not `base64`).
- **cache_utils.py** — progressive disclosure cache for large outputs.

## Testing: recorded fixtures are mandatory

**The defining bug class in this repo is code and tests both written against
*imagined* tool output.** When the imagination is wrong they are wrong in the
same direction, so the suite stays green while the script does nothing. Three
advertised capabilities shipped inert this way, past 470 passing tests.

Therefore:

1. **Parser tests read `tests/fixtures/recorded/`; they never inline tool output
   as a string literal.** Use the `recorded` pytest fixture. If ground truth is
   missing, record it — do not hand-write a plausible substitute.

   ```bash
   python tests/record_fixtures.py --list
   python tests/record_fixtures.py --only logcat_threadtime
   python tests/record_fixtures.py --profile pixel4xl-api33 --serial <SERIAL>
   ```

   Fixtures are namespaced per device under `tests/fixtures/recorded/<profile>/`,
   so the same command captured on a different API level is a new file rather
   than a silent overwrite — diffing two profiles is how you spot a format that
   shifted between releases. Use the `any_profile` pytest fixture for invariants
   that must hold everywhere ("this subcommand does not exist"), and `recorded`
   for the complete primary profile.

   **Never commit a fixture containing private data.** The recorder aliases
   non-AOSP package names to `com.example.appN`, but that does not catch
   free-text content. When recording from a personal device, capture only
   structural output (`wm size`, `cmd ... help`) — not logcat, notification
   lists, or screen dumps.

2. **Before writing any code that parses a tool's output, run the tool and look
   at it.** Several commands in this codebase do not exist at all
   (`cmd statusbar battery-level`, `dumpsys activity anr`,
   `cmd notification list channels`). Some fail silently and exit 0.

3. **The rule is enforced, not just written here.** `tests/test_fixture_policy.py`
   is a ratchet: it fails any *new* test that feeds a long string literal into a
   parser, either directly or through a mocked `subprocess`'s `stdout=`. The
   violations that exist today are frozen in `KNOWN_VIOLATIONS` as debt, and a
   stale entry fails too, so paying one off cannot silently re-open the door.

   `test_container.py` and `test_model_inspector.py` were the two worth paying
   off first, and they are paid: both now read the real thing from
   `tests/fixtures/recorded/` — `run_as_ls_data_dir`, `run_as_ls_databases`,
   `run_as_not_an_application`, `shared_prefs_settings_xml`,
   `sqlite_schema_host`.

   Read the current debt off `KNOWN_VIOLATIONS`, not off this paragraph. The
   frozen set is the copy that fails when it is wrong; a prose list of what is
   outstanding goes stale silently, which this one did.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fluxxion82/android-emulator-skill](https://github.com/fluxxion82/android-emulator-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-02 -->

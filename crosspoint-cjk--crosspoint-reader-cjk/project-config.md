---
trigger: always_on
description: Development instructions for Claude and other assistants in `crosspoint-reader-cjk`.
---

# CLAUDE.md

Development instructions for Claude and other assistants in `crosspoint-reader-cjk`.
Read `AGENTS.md` first; it is the authoritative repository guide. This file highlights the rules most likely to be missed during automated work.

## Repository identity

This is the CJK-focused CrossPoint Reader fork, not a stock upstream checkout. It targets memory-constrained Xteink ESP32-C3 hardware and also keeps the ESP32-S3 `sticky` target buildable. Upstream code may be a useful baseline, but upstream behavior does not automatically supersede this fork's tested behavior.

Before work begins:

```bash
git branch --show-current
git remote -v
git status --short
git submodule status --recursive
```

Preserve unrelated changes. Do not assume branch or remote names. Never rewrite shared history, force-push, or perform destructive device flashing without explicit authorization.

## Submodules are build inputs

Initialize recursively:

```bash
git submodule update --init --recursive
```

- `freeink-sdk/` is mandatory. `platformio.ini` references its libraries with local `symlink://` dependencies; a non-recursive or missing checkout will not produce a valid build.
- `freeink-sdk/libs/assets/Icons/lucide` is a nested source asset submodule used when regenerating Lucide-derived icons. Ordinary compilation can use the existing generated icon headers, but recursive initialization keeps the SDK checkout complete and matches CI.
- The recorded SDK gitlink belongs to this fork. Do not silently advance it to another SDK branch or upstream HEAD.
- Verify uncertain display, storage, network, board, and input APIs against the checked-out SDK source before using them.
- SDK implementation changes and the main-repository gitlink update should be deliberate and separately reviewable.

## Build environments

Use the correct environment:

- `default`: normal ESP32-C3 development firmware; EN/SC/TC/JA.
- `gh_release` / `gh_release_rc`: Simplified Chinese release line; EN/SC/JA.
- `gh_release_tc` / `gh_release_rc_tc`: Traditional Chinese release line; EN/TC/JA.
- `slim`: C3 size-focused build without serial logging.
- `device_test`: device automation with serial input injection; test-only and never a release artifact.
- `recovery`: minimal English SD recovery firmware with a separate source filter.
- `sticky`: ESP32-S3 target used by CI; it is not an X4-compatible binary.

Useful commands:

```bash
./bin/clang-format-fix -g
pio check --fail-on-defect low --fail-on-defect medium --fail-on-defect high
pio run -e default -e sticky
pio run -e gh_release
pio run -e gh_release_tc
pio run -e recovery
pio run -e default --target upload
pio device monitor
```

CI intentionally builds `default` and `sticky` in one `pio` invocation. Reproduce that shape when checking CI failures because separate invocations can invalidate artifacts as toolchains change.

For Xteink C3 uploads, retain the registered safe upload path. It writes application partitions and updates OTA selection while preserving the bootloader, partition table, and data partitions such as NVS, SPIFFS, and coredump; it does not preserve old factory/OTA application images. After testing `device_test` or temporary screenshot configurations, remove temporary overrides and restore `default` firmware.

`platformio.local.ini` is machine-local and gitignored. Never commit it. Do not use `git clean -fdX`; it can remove this file and other intentionally ignored assets.

## Generated sources

PlatformIO runs these pre/post scripts:

- `scripts/patch_wolfssl.py`
- `scripts/build_html.py` (walks `src/`; generates adjacent headers from Web `.html` and `.js` inputs)
- `scripts/gen_i18n.py`
- `scripts/gen_builtin_cjk_font.py`
- `scripts/git_branch.py`
- `scripts/patch_jpegdec.py`
- `scripts/register_unit_tests_target.py`
- `scripts/register_safe_upload.py`

Do not hand-edit generated outputs:

- `src/**/*.generated.h`: edit the adjacent Web `.html` or `.js` input.
- `lib/I18n/I18nKeys.h`, `I18nStrings.h`, `I18nStrings.cpp`: edit `lib/I18n/translations/*.yaml` or the generator.
- `lib/GfxRenderer/cjk_ui_font_*.h`: edit the tracked font inputs/generation scripts and regenerate.
- `lib/Epub/Epub/hyphenation/generated/*`: change the source/generator path.

The active PlatformIO environment filters translation tables and the CJK glyph corpus. An i18n change that builds only `default` is insufficient release verification: build both SC and TC release environments and check generated glyph coverage. Those environment-specific builds rewrite tracked CJK headers, so finish by regenerating/restoring the canonical all-shipping-language headers (for example with `python3 scripts/gen_builtin_cjk_font.py`) and run `python3 scripts/check_cjk_ui_font_charset.py` before committing.

## Fork preservation contract

Read `docs/fork-features.md` before an upstream merge or a refactor in a high-risk subsystem. Preserve behavior, not merely symbol names.

### Fonts and catalog

- Reader SD font and UI SD font are independent persisted selections and runtime roles.
- Official UI packages contain physical 8, 10, and 12 pt faces. Load UI faces 12 -> 10 -> 8; incomplete manually installed families may use the nearest size within the same family.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CrossPoint-CJK/crosspoint-reader-cjk](https://github.com/CrossPoint-CJK/crosspoint-reader-cjk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->

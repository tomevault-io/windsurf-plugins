---
trigger: always_on
description: 如果你是参与项目的LLM，请维护此AGENTS.md，描述你修改的内容，添加或修改的东西，项目重点，TODO等：
---

详细说明请参考
V4说明.md

如果你是参与项目的LLM，请维护此AGENTS.md，描述你修改的内容，添加或修改的东西，项目重点，TODO等：

Updates:
- Split the root and Ver4.0 bilingual READMEs into dedicated English `README.md` and Chinese `README_zh.md` files, with bidirectional language navigation matching the HVCCPS V1.4 repository.
- Converted all README image links to full GitHub raw URLs for the current repository docs.
- Fixed LCD 7x5 glyph row mapping in `hardware/1601a.c` so both normal and inverted mount are top-aligned on 8-row LCD cells and no longer crop the glyph last row.
- Fixed config-mode confirmation prompt rendering to avoid custom-glyph overflow artifacts (e.g., `RESTORE?`/`#=Y *=N`) by using compact in-font prompts.
- Added custom LCD glyph support for `/` so prompts like `SAVE?#/*` render fully without missing slash.
- Tuned `/` glyph stroke width for better visibility on LCD1601A so `SAVE?#/*` is readable under both mount orientations.
- Added bilingual user-facing step-by-step config-mode tutorial (with example) at the top of `Ver4.0/config.md`.
- Refined config tutorial wording (CN/EN) to clarify users can modify settings directly in config mode without re-uploading firmware.
- Added configurable pre-explosion beep stage: on explosion entry, LED turns full yellow, buzzer sounds at 3600Hz for `CONFIG_EXPLOSION_BEEP_MS`, then continues original explosion flow.
- Added configurable arm wrong-password hint: when preset arm password is enabled, wrong input can now show `ERROR` via `CONFIG_ARM_ERROR_HINT_ENABLE`; default remains silent clear for backward behavior.
- Simplified config sources: removed `config_defaults.h` and switched runtime default loading to `config.h` as the single source of truth.
- Simplified docs: merged CN/EN config explanations into `Ver4.0/config.md` with per-item config IDs beside each setting description.
- Removed `default_config_CH.h` and `default_config_EN.h`, and updated README docs to point users to `config.md` + `config.h` only.
- Updated runtime config compatibility to include a defaults checksum so reflashing with changed `config.h` defaults applies immediately without requiring manual restore.
- Added a full English section to `Ver4.0/README.md` and converted all file/image links to absolute GitHub URLs.
- Refined Ver4.0/README.md with clean CN/EN formatting and direct folder links in the soldering/assembly section.
- Updated Ver4.0/README.md with in-section images, direct config links, quick-start tuning items, and MIT badge logo.
- Updated Ver4.0/V4说明.md to align behavior with current config macros and removed outdated TODOs.
- Clarified 3DP_Models readme wording and link to the model provider for print requests in CN/EN.
- Reformatted `Ver4.0/3DP_Models/readme.md` with clearer wording and bilingual (CN/EN) sections.
- Updated root README with V4 image, repository-scope maintenance notice, and bilingual intro with English jump anchor.
- Added LCD normal-mount config and moved orientation handling (column mapping + glyph rotation) into the 1601a driver so app logic stays logical-only.
- Reworked countdown scroll rendering to draw stars at i-1/i/i+1 within bounds using a single moving position.
- Added symmetric scroll blur ramping at both edges and a 3-beep defuse-success tone using the startup frequency.
- Adjusted countdown scroll range to 0..mid screen with edge-length ramping (* / ** / ***) and reworked defuse-success to blink stars vs code without backlight flashing.
- Added password-specific LCD write path to avoid CGRAM slot reuse flicker during defuse animation.
- Rewrote Ver4.0/README.md to fully include V4说明.md content with normalized language, condensed code, and a clear architecture overview.
- Renamed LCD driver to 1601a.c/1601a.h, added backlight control, updated Keil project references.
- Added config constants in user/config.h and wired timing/ratios (buzzer 3600 Hz, scroll speed, beep formula, LED yellow ratio).
- Updated countdown scroll to "***" with faster sweep and adjusted beep timing logic.
- Manual/external defuse now cancels on release and resets animation; success effect flashes LED + LCD backlight and shows code.
- Clarified MP3 command mapping for power-on, defuse success, explosion effect, and music box track.

- Split startup vs countdown buzzer frequencies, added PWM backlight level, and added MP3 enable/selection config; mp3_over now plays on arming and explosion audio is single-choice.

- Added buzzer duty control, synchronized defuse-success buzzer flashing, and expanded config documentation with min/max/typical values.

- Switched buzzer duty calculation to use TIM3->ARR to avoid missing TIM_GetAutoreload symbol.

- Rewrote config comments as bilingual, aligned min/max/typical formatting, and added compile-time range checks with CONFIG_IGNORE_RANGE.

- Converted beep interval config to millisecond integers, fixed countdown timing math, and rewrote config comments in GBK for readable Chinese.

- Moved range checks to config_range.h, switched configs to UTF-8 BOM for readable Chinese, and removed max limits where not required.

- Added config toggles for password/manual/external defuse modes, plus optional preset password requirement for arming.
- When preset arm password is enabled, a wrong entry now clears the input back to stars.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AzidoPP/STM-C4](https://github.com/AzidoPP/STM-C4) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->

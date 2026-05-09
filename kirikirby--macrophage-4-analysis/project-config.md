---
trigger: always_on
description: - Use English for all new narrative content in this file (AGENTS.md).
---

# AGENTS

## Language Requirement
- Use English for all new narrative content in this file (AGENTS.md).
- Quoted strings and code/comment examples may remain in their original language.

These instructions apply to this repository.

## Encoding / Character Set Requirements
- All repository text files must remain UTF-8 encoded.
- Do not introduce emoji, dingbats, box-drawing characters, or other decorative symbols.
- Prefer ASCII for structural markers in logs/UI text (examples: "OK", "WARN", "X", "|-", "-").
- CJK characters and standard punctuation already used in the script are allowed.

## IMPORTANT: Script Module Index (Update Required)
- This repository maintains a module index for `Macrophage Image Four-Factor Analysis_4.0.0.ijm`.
- Every code change that adds/moves/removes logic must update the index line numbers below.
- Keep the index in sync so future AI can locate modules quickly.

## IMPORTANT: Error Code System (Update Required)
- Every user-facing error message must include a stable error code in the format `[E###]` at the start of the message text.
- Error codes must be kept consistent across CN/JP/EN strings for the same failure.
- When an error is shown or causes an exit, log the error with the code (use the structured log style and `T_log_error`).
- Any new failure points introduced by new features or optimizations must add:
  - A new error code and localized CN/JP/EN messages.
  - A log entry that includes the code.
  - A corresponding entry in the Error Code Index below.
- Validation errors inside dialogs should be recoverable (prompt the user to correct input and continue), not hard-exit the script unless the failure is truly fatal.

## Error Code Index (Macrophage Image Four-Factor Analysis_4.0.0.ijm)
- E001: Required window missing (requireWindow).
- E002: Image open failed (openImageSafe).
- E003: Too many cell ROIs (>65535) for label mask.
- E004: ROI[1] invalid bounds (label mask generation).
- E005: Label mask fill check failed (center pixel still 0).
- E006: Selected folder mixes files and subfolders.
- E007: Nested subfolders detected (recursive structure not supported).
- E008: No image files found in the selected folder.
- E009: ROI list empty when attempting to save.
- E010: ROI zip save failed (file not created).
- E011: ROI zip load failed or contains no valid ROI.
- E012: Feature selection conflict (Feature 1 + Feature 5).
- E013: No feature selected.
- E020: Feature reference image unavailable or timed out.
- E101: Filename rule empty.
- E102: Filename rule format invalid (split failed).
- E103: Filename rule parts missing.
- E104: Filename rule tokens invalid (only <p>/<f> allowed).
- E105: Filename rule must include both <p> and <f>.
- E106: Filename rule order invalid.
- E107: Subfolder-keep mode requires folderRule//fileRule.
- E108: Subfolder rule not allowed in current mode.
- E109: Double slash appears more than once.
- E110: Rule parameters must be key="value".
- E111: Unknown rule parameter prefix.
- E112: Rule parameter value must be in English double quotes.
- E113: Invalid f parameter value (must be "F" or "T").
- E114: Duplicate f parameter in rule spec.
- E115: Quoted literals are not supported in filename rules.
- E121: Column format empty.
- E122: Column format contains empty item.
- E123: Column format contains empty token.
- E124: Column parameters must be comma-separated.
- E125: "$" missing column code.
- E126: "$" used on built-in column.
- E127: Column parameters must be key="value".
- E128: Unknown column parameter prefix.
- E129: Column parameter value must be in English double quotes.
- E130: Unknown column token.
- E131: Column parameter name is empty.
- E132: Column parameter value is empty.
- E133: Duplicate column parameter key.
- E134: Custom column missing name/value parameter.
- E135: Multiple "$" custom columns specified.
- E141: Fluorescence prefix empty.
- E142: Fluorescence prefix contains invalid path separator.
- E143: No fluorescence images found with the specified prefix.
- E144: No target fluorescence color samples selected.
- E145: No near/halo fluorescence color samples selected.
- E146: Fluorescence RGB format invalid.
- E147: Fluorescence RGB range invalid.
- E148: Exclusion colors enabled but none provided.
- E149: Fluorescence image size mismatch vs normal image.
- E201: Non-numeric value entered in numeric parameter dialog fields.
- E202: Parameter spec format invalid.
- E203: Parameter spec unknown or duplicate key.
- E204: Parameter spec missing required key.
- E205: Parameter spec value invalid.
- E206: Tuning repeat count invalid.
- E207: Fluorescence tuning requires at least two time points with fluorescence images.
- E208: Fluorescence tuning could not compute valid eTPC/#eTPC pairs.
- E199: Data formatting validation error fallback (missing code in message).

### Module Index (Macrophage Image Four-Factor Analysis_4.0.0.ijm)
- Header + settings: `Macrophage Image Four-Factor Analysis_4.0.0.ijm:1`
- Log + math utilities: `Macrophage Image Four-Factor Analysis_4.0.0.ijm:72`
- File/string/CSV helpers: `Macrophage Image Four-Factor Analysis_4.0.0.ijm:161`
- Token/rule parsing + data-format validation: `Macrophage Image Four-Factor Analysis_4.0.0.ijm:2504`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KiriKirby/Macrophage-4-Analysis](https://github.com/KiriKirby/Macrophage-4-Analysis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->

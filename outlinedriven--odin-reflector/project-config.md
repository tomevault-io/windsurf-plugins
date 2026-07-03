---
trigger: always_on
description: This file is only for repo-specific constraints that are easy to break and expensive to rediscover. README owns installation and user-facing usage; do not duplicate it here.
---

# Codex Reflector Agent Notes

This file is only for repo-specific constraints that are easy to break and expensive to rediscover. README owns installation and user-facing usage; do not duplicate it here.

## Surface split

- **Rule:** Treat the repository as two hook surfaces, not one implementation. The Claude/Cursor plugin is `scripts/codex-reflector.py` wired by `hooks/hooks.json`; the oh-my-pi port is `omp/codex-reflector.ts` loaded through `package.json` `omp.extensions`.
  **Why:** Both surfaces call `codex exec` and expose the same reflector idea, but their hook delivery and stop-enforcement mechanisms differ. Editing the wrong surface fixes nothing.

- **Rule:** For shared behavior changes, check both surfaces before declaring parity: verdict parsing, prompt builders, redaction/sandboxing, stop-review enforcement, model/effort gating, and changed-file target resolution.
  **Why:** The TypeScript file is a native port of the Python hook, while the OMP tests codify port-specific contracts. Silent drift produces different review outcomes for Claude/Cursor vs OMP users.

- **Rule:** Any shipped OMP or Claude plugin code/config change bumps the paired release manifests in the same commit: the OMP version (`package.json` `version`) and the Claude plugin version in all three Claude fields (`.claude-plugin/plugin.json` `version`, `.claude-plugin/marketplace.json` top-level `version`, and `marketplace.json` `plugins[0].version`); also refresh any marketplace `metadata.lastUpdated` date present. Shared Python hook changes also bump the Cursor plugin version in all three Cursor fields (`.cursor-plugin/plugin.json` `version`, `.cursor-plugin/marketplace.json` top-level `version`, and `marketplace.json` `plugins[0].version`).
  **Why:** The OMP and Claude surfaces ship independently but evolve together, so bumping only one side hides paired release changes from users inspecting either manifest. The Python hook code also powers Cursor; Cursor manifests are tracked source, but they should move when the shared Python/Cursor surface changes rather than on OMP-only changes.

## Python Claude/Cursor plugin invariants

- **Rule:** Keep `hooks/hooks.json` as routing glue and keep real classification in `classify()` inside `scripts/codex-reflector.py`. Cursor-specific matcher generation belongs in `scripts/install-cursor.sh`, not in the core dispatch path.
  **Why:** Claude and Cursor expose different hook payloads and matcher behavior. A duplicated routing table becomes a drift source; the Python script normalizes payloads and owns the routing decision.

- **Rule:** Python blocking is exit-code based. Advisory reviews exit `0` with JSON; Stop blocks by returning `decision: "block"`/`_exit: 2`, which `main()` emits on stderr before exiting `2`.
  **Why:** Claude consumes exit `2` stderr as blocking context. Treating Stop like PostToolUse JSON feedback either fails schema validation or silently approves work that should block.

- **Rule:** `hookSpecificOutput` is only for events whose schema accepts it (`PostToolUse`, `PostToolUseFailure`, `PreToolUse`, `UserPromptSubmit`). Stop must use `systemMessage` or `decision`/`reason`; PreCompact must use `systemMessage`.
  **Why:** Stop/PreCompact reject `hookSpecificOutput`; putting it there breaks the hook response instead of injecting useful context.

- **Rule:** Parse verdicts from raw Codex output before any compaction in every responder that branches on PASS/FAIL/UNCERTAIN.
  **Why:** Compaction can remove or rewrite the verdict line. A buried or stripped verdict becomes UNCERTAIN and changes fail-open/fail-closed behavior.

- **Rule:** The Python plugin is stateless — no `.json` FAIL cache and no `fcntl` state file. `respond_code_review`/`respond_plan_review` inject the verdict + opinion inline as `systemMessage` (every verdict), adding `hookSpecificOutput.additionalContext` for FAIL/UNCERTAIN. `respond_stop` is a fresh holistic review run once per stop chain: only a FAIL blocks (`decision: "block"`/`_exit: 2`); PASS and UNCERTAIN settle via `systemMessage` (fail-open — never block on uncertainty), and the retained `stop_hook_active` guard settles a re-stop.
  **Why:** Per-tool reviews self-correct inline; the holistic Stop review is the gate. Python has no continuation cap, so `stop_hook_active` is the only safe loop bound — re-reviewing every stop without it could never settle.

- **Rule:** Keep Cursor payload adaptation contained in `_normalize_cursor_input()` and generated settings from `scripts/install-cursor.sh`.
  **Why:** Cursor compatibility maps event names and fields into Claude-shaped hook data. Scattering Cursor field handling through responders makes every future hook change harder to audit.

## OMP native extension invariants

- **Rule:** The OMP surface is a default-export hook factory. `CODEX_REFLECTOR_ENABLED=0` must register no handlers.
  **Why:** OMP loads the extension through the manifest/factory contract; the kill switch must be safe even when the package is present.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OutlineDriven/odin-reflector](https://github.com/OutlineDriven/odin-reflector) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-03 -->

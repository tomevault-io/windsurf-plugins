---
trigger: always_on
description: <!-- DO NOT REGENERATE via /init — manually maintained -->
---

<!-- DO NOT REGENERATE via /init — manually maintained -->

# CLAUDE.md

> **Attribution.** The four behavioral rules below ("Think Before Coding", "Simplicity First", "Surgical Changes", "Goal-Driven Execution") are reproduced **verbatim** from Andrej Karpathy's [`CLAUDE.md`](https://raw.githubusercontent.com/forrestchang/andrej-karpathy-skills/main/CLAUDE.md), distributed via the `forrestchang/andrej-karpathy-skills` repository (transferred to [`multica-ai/andrej-karpathy-skills`](https://github.com/multica-ai/andrej-karpathy-skills) in May 2026). Used verbatim with attribution; no edits to wording. Project-specific rules follow under **`## BFL Project Rules`**.

Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

---

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.

---

## BFL Project Rules

This is a **ComfyUI custom-nodes package** that wraps the **Black Forest Labs (BFL) FLUX** image-generation API. Single Python package, ~1000 LOC, no test framework, ruff for lint/format.

### Always do

- **Inheritance**: every new generation node inherits from `BaseFlux` (`nodes/base.py`). Every finetune node inherits from `BaseFinetuneFlux` (same file). The base class owns `post_request` / `get_result` / `process_result` / `create_blank_image` / `check_multiple_of_32` — do **not** reimplement those.
- **Node registration**:
  - Python class name is bare (e.g., `FluxPro11`, `FluxDev`). The one exception is `FluxConfig_BFL` where the class name matches its registry key for historical reasons.
  - ComfyUI **registry key** in `NODE_CLASS_MAPPINGS` ends with `_BFL`: `"FluxPro11_BFL": FluxPro11`.
  - ComfyUI **display name** in `NODE_DISPLAY_NAME_MAPPINGS` ends with ` (BFL)`: `"FluxPro11_BFL": "Flux Pro 1.1 (BFL)"`.
  - `CATEGORY` is one of `"BFL"` (generation), `"BFL/Finetune"`, `"BFL/Utility"`, `"BFL/Utils"`, `"BFL/Config"`. Match the existing module's choice.
  - A brand-new module file under `nodes/` is invisible until its name is appended to `node_list` in `__init__.py:4-9`.
- **Logging**: every outbound request and every status line is prefixed `[BFL] ` so it's grep-able in the ComfyUI console.
- **curl-equivalent**: every POST prints the full curl-equivalent (URL, headers, body) before sending. Matches the pattern in `nodes/base.py:60-62`. Useful for users debugging the BFL API directly.
- **Width / height validation**: when a node takes `width` / `height` ints, the request path runs `self.check_multiple_of_32(width, height)` (already called by `BaseFlux.generate_image` if they're in `arguments`). Newer Flux 2 / Ultra models use `aspect_ratio` strings (`"16:9"`, `"1:1"`, …) and bypass dimension validation.
- **Seed handling**: `seed = -1` means *random*. The node parameter defaults to `-1`. Only insert `arguments["seed"] = seed` into the request body when `seed != -1` — never send `-1` to BFL.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gelasdev/ComfyUI-FLUX-BFL-API](https://github.com/gelasdev/ComfyUI-FLUX-BFL-API) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->

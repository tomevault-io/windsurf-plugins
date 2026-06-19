---
trigger: always_on
description: Use when the user asks to generate or edit images/videos with Alibaba Cloud Bailian/DashScope Wan 2.7 or Wan video models, including 文生图, 图生图, 文生视频, 图生视频, 参考生视频, 视频编辑, or 通义万相.
---


# Wan 2.7

Use this skill to execute Wan image and video generation requests, not to merely explain the API.

## What this skill does

- Supports text-to-image.
- Supports image-to-image with one or more local files or public URLs.
- Supports text-to-video through Wan video models.
- Supports wan2.7 image-to-video with `media` inputs:
  - first frame
  - first frame + last frame
  - first clip continuation
  - optional driving audio
- Supports structured video specs for:
  - reference-to-video
  - VACE video editing
  - legacy first/last-frame image-to-video
- Supports chat-side interactive portrait prompt building backed by local carrier / trigger data.
- Supports the V1 demo parameter set:
  - `size`
  - `n`
  - `seed`
  - `watermark`
  - `enable_sequential`
  - `thinking_mode`
  - `color_palette`
  - `bbox_list`
- Saves request/response artifacts and downloaded image or video outputs locally.
- Returns local file paths so the caller can inline-display generated images or link generated videos.

## Prompt construction

Do not treat all prompts as the same shape.

Portrait-centric requests need a separate chat-side workflow before the final prompt is written.

The official Wan 2.7 image-edit doc describes a few distinct usage patterns:

- general instruction-following edits
- multi-image fusion
- subject-feature preservation
- precise local editing with `bbox_list`
- multi-panel / continuous output for wan2.7 models

This local skill also adds one structured chat-side pattern:

- interactive portrait construction using local carrier / trigger data

For this local skill, choose a prompt task type first, then write the prompt with the matching template.

Canonical reference:

- [prompt-task-types.md](F:/Documents/Playground/wan2.7-image-demo/references/prompt-task-types.md)
- [video-prompt-task-types.md](F:/Documents/Playground/wan2.7-image-demo/references/video-prompt-task-types.md)
- [chat-onboarding-flow.md](F:/Documents/Playground/wan2.7-image-demo/references/chat-onboarding-flow.md)
- [video-api-notes.md](F:/Documents/Playground/wan2.7-image-demo/references/video-api-notes.md)
- [video-runner-spec.md](F:/Documents/Playground/wan2.7-image-demo/references/video-runner-spec.md)
- [portrait-chat-flow.md](F:/Documents/Playground/wan2.7-image-demo/references/portrait-chat-flow.md)
- [portrait-data-schema.md](F:/Documents/Playground/wan2.7-image-demo/references/portrait-data-schema.md)
- [triggers_by_dim.json](F:/Documents/Playground/wan2.7-image-demo/references/portrait-data/triggers_by_dim.json)
- [carriers.json](F:/Documents/Playground/wan2.7-image-demo/references/portrait-data/carriers.json)

Hard boundaries:

- Do not invent hidden prompt-rewrite behavior. Image tasks do not expose `negative_prompt` or `prompt_extend`; video tasks may expose model-specific `negative_prompt` or `prompt_extend` through the video runner.
- Do not claim dedicated detection/segmentation task support in this runner until the request/response contract is verified locally.
- For multi-image input, explicitly assign each image a role in the prompt.
- For editing tasks, state preservation constraints before the requested change.
- Portrait flow is a chat-side overlay, not a new runner mode.
- Do not dump every available portrait trigger into one message.
- Do not override `carrier.fixed` fields in place; switch to a compatible carrier instead.
- For video tasks, use public HTTP/HTTPS media URLs. The local video runner does not upload local files.

## Portrait prompt workflow

When the user wants any human-centered result and face / age / ethnicity / hairstyle / expression are primary controls, switch into the portrait workflow before writing the final prompt.

This is mandatory.

If the primary subject is a person, do not jump directly from the user's sentence to a freehand final prompt.
Even when the user already gave a detailed description, you must still trigger the portrait workflow:

- pre-fill the slots already implied by the request
- ask only the unresolved or high-impact slots
- then assemble the final descriptor from carrier data

Typical triggers:

- `人物`
- `人像`
- `证件照`
- `头像`
- `avatar`
- `headshot`
- `保持这个人的长相不变`

Use the local portrait data files as the source of truth:

- `references/portrait-data/triggers_by_dim.json`
- `references/portrait-data/carriers.json`
- `references/portrait-data-schema.md`

Workflow rules:

1. Start with build depth:
   - `快速成型`
   - `标准捏脸`
   - `深度定制`
2. Recommend 2-4 compatible carriers based on carrier data, in this order:
   - `use_cases`
   - `build_depth`
   - `fixed`
   - `slots` count only as a tiebreaker
3. Treat `carrier.fixed` as hard constraints. If the user wants conflicting traits, change carriers instead of editing fixed fields.
4. Walk `carrier.slots` in order, one slot at a time.
5. Resolve slot options this way:
   - no `slot_source` -> `triggers_by_dim[slot]`
   - `slot_source.kind = dim_union` -> ask category first, then concrete values
   - `slot_source.kind = literal_values` -> use the literal values directly
   - `slot_source.kind = filtered_dim` -> use the declared subset from the referenced dimension

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sdbds/Wan-2.7-Skill](https://github.com/sdbds/Wan-2.7-Skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

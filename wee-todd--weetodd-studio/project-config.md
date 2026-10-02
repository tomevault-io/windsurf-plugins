---
trigger: always_on
description: WeeTodd Studio is a standalone macOS creative app for Apple Silicon. The Swift app is the primary
---

# WeeTodd Studio agent guide

WeeTodd Studio is a standalone macOS creative app for Apple Silicon. The Swift app is the primary
product; ComfyUI nodes remain a maintained secondary interface to the shared renderer and media
utilities. Draw Things integration, native MLX inference, creative planning and reusable production
assets belong to the same product.

## Product direction

- Prioritize standalone-app usability, performance and complete creative workflows. ComfyUI is
  not a prerequisite for Studio; preserve node contracts and workflow compatibility as shared
  capabilities evolve.
- Treat Draw Things as a central integration and the preferred starting point for fast inference
  when it supports the requested model/task. Its connection helper can remain an optional runtime
  dependency. Distinguish local/self-hosted inference, Cloud API execution and native model reuse.
- Native inference provides additional models and advanced tasks, including supported A2V/V2V,
  conditioning and controls beyond the Draw Things integration. Current native engines are H3,
  LTX 2.3 and LTX 2.5. Consider additional models when they close a useful feature gap.
- Prefer sharing compatible installed weights in place, including supported Draw Things model
  files, before requiring another download/conversion. Compatibility is per component and task;
  model reuse does not imply Draw Things sampler parity or support for every native task.
- New Draw Things models should fit the existing discovery/interface where possible. Verify their
  capability mapping, settings and transport before claiming support; model discovery alone is
  insufficient. Keep model/backend limitations visible to users and retain useful native features.
- Make native generation feel familiar to Draw Things users: expose model/task selection, supported
  sampling parameters, seeds, conditioning, LoRAs and reusable LoRA groups through consistent Studio
  controls. Routine generation should not require editing or importing a JSON recipe.
- Give native and Draw Things LoRA stacks consistent strength, enable/disable and group-application
  interactions while preserving backend-specific file formats, model/task compatibility and sampling
  requirements. Show which settings a specialized adapter such as Turbo requires before applying it.
- Keep advanced execution details available without crowding ordinary generation controls. Director
  and manual generation must resolve through the same validated settings and retain reproducible takes;
  a familiar control must never silently accept a setting the selected engine cannot execute.
- Product/repository name: WeeTodd Studio / `wee-todd/WeeTodd-Studio`. Keep existing package names,
  import modules, node IDs, document formats and saved local paths compatible unless a task
  explicitly includes a tested migration. Do not rename a user's checkout or model library as a
  side effect of the product rename.

## Scope boundary

- Supported scope includes Studio editing and creative planning, Draw Things image/video integration,
  shared model storage, native MLX engines, maintained ComfyUI nodes and relevant media utilities.
  New model integrations must serve the product direction and the assigned task.
- Keep native engines, local planning and optional remote execution behind their existing adapters.
  Do not expand an assigned change into unrelated application or account functionality.
- Independently implement and test native algorithms researched from third-party implementations.
  Optional runtime dependencies remain separate and retain their licenses and distribution limits.
- Never commit model weights, outputs, caches, tokens, credentials, or machine-specific paths.

## Development rules

- Keep node imports lightweight; load MLX weights only when a graph executes.
- Keep the H3 and LTX engines isolated behind separate adapters. Studio and headless jobs must
  reuse the shared renderer without importing ComfyUI or maintaining a second sampler.
- Before changing an LTX 2.5 loader, sampler, VAE, conditioning contract, or optimization default,
  compare current Lightricks LTX-2 releases, LTX-2.5 checkpoint files, and native ComfyUI changes
  against the baseline in `docs/reference/LTX25_MLX_INTEGRATION.md`. Update the baseline and OKF log
  when upstream changes. Do not transfer CUDA performance claims to MLX without measurement.
- Preserve synchronized audio and video as a single H3 generation contract.
- Keep model state process-local and explicitly unloadable.
- Default weighted stages to staged unloading: Qwen3-VL, transformer, video VAE, then audio VAE.
  Keep a component warm only through an explicit node control and report its resident state.
- Release the active component after success, failure, or cancellation when staged unloading is
  selected. Do not load the next weighted stage before the prior stage is releasable.
- Validate dimensions, duration, checkpoint paths, and task support before expensive work.
- Add a focused test for every node contract or engine behavior changed.
- Treat portable workflow paths and runtime-ready workflow paths as separate validation states.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wee-todd/WeeTodd-Studio](https://github.com/wee-todd/WeeTodd-Studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->

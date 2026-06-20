---
trigger: always_on
description: >
---


# BananaHub

Generate or edit images from non-English or mixed-language requests inside one `/bananahub` workflow. GPT Image 2 through an OpenAI-compatible image endpoint is the default provider path; user-configured Gemini/Nano Banana, OpenAI official, Vertex, chat-compatible paths, and Codex/host-native image tools are preserved. BananaHub keeps prompt optimization, conservative enhancement, model fallback, image editing, template use, and BananaHub discovery in a single skill instead of splitting them across separate installs.

When BananaHub is loaded implicitly for a generic agent image request, act as a lightweight optimization layer first. Do not silently take over generation: ask whether the user wants BananaHub to optimize the prompt before the image tool runs, then respect the answer.

## Quick Start

- Install via Open Agent Skills: `npx skills add https://github.com/bananahub-ai/bananahub-skill --skill bananahub`
- Install in Claude Code directly: `claude skill install https://github.com/bananahub-ai/bananahub-skill`
- Run setup once: `/bananahub init`
- Test a host-native image tool such as Codex built-in image generation: `/bananahub test-host-imagegen`
- Generate from a natural-language request: `/bananahub 一只橘猫趴在键盘上打盹`
- Edit an image: `/bananahub edit 把背景换成海滩 --input photo.png`
- Discover a reusable template: `/bananahub discover 代码库讲解图`
- Capture the current image iteration as a workflow template: `/bananahub capture-workflow`

## Key Paths

- **Generation script**: `{baseDir}/scripts/bananahub.py`
- **Provider adapters**: `{baseDir}/scripts/providers/` — Gemini, OpenAI Images, and chat/completions-compatible runtime adapters
- **Runtime config module**: `{baseDir}/scripts/runtime_config.py` — provider constants, aliases, transport defaults, config keys, and endpoint normalization
- **Config store module**: `{baseDir}/scripts/config_store.py` — config loading, profile merge, validation, provider override, and serialization helpers
- **Prompt optimization rules**: `references/prompt-guide.md` — read during Phase 1 (base optimization)
- **Enhancement profiles**: `references/profiles/{name}.md` — read during Phase 3 (on-demand)
- **Official references**: `references/official-sources.md` — authoritative source URLs, core example library
- **Capability registry**: `references/capability-registry.md` — provider/model feature routing and fallback policy
- **Model registry**: `references/model-registry.json` — canonical model ids, aliases, defaults, and provider families
- **Provider guides**: `references/providers/{provider}.md` — lazy-loaded model-family prompt and runtime rules
- **Template system**: `references/template-system.md` — read when handling templates/use/create-template commands
- **Hub discovery guide**: `references/hub-discovery.md` — read when handling `discover` or when local template matching is weak
- **Template files**: `{baseDir}/references/templates/<id>/template.md` (built-in) + `~/.config/bananahub/templates/<id>/template.md` (user-installed)
- **Telemetry helper**: `python3 {baseDir}/scripts/bananahub.py telemetry ...` — use for built-in/installed template adoption events
- **Telemetry state**: `~/.config/bananahub/telemetry.json` — stores the local anonymous usage id
- **Init guide**: `references/init-guide.md` — read when handling `init` command
- **Optimization pipeline**: `references/optimization-pipeline.md` — read when optimizing prompts
- **Template format spec**: `references/template-format-spec.md` — detailed field definitions, repo structure, sample requirements
- **Template validator**: `python3 {baseDir}/scripts/validate_templates.py` — validates bundled/user template metadata for schema v1/v2 compatibility
- **Mode detector**: `python3 {baseDir}/scripts/bananahub.py check-mode` — reports provider-backed / host-native / prompt-only execution mode and capability layer boundaries
- **Host-native image tool test**: `/bananahub test-host-imagegen` or `/bananahub test-codex-imagegen` — skill-layer command; call the host/Codex built-in image generation tool with the validation prompt below
- **Prompt archive**: current working directory `bananahub-prompts/` when `--save-prompt`, `--prompt-output`, or `BANANAHUB_SAVE_PROMPTS=1` is used
- **API config** (priority high→low):
  1. `--config <file>` CLI flag
  2. Selected skill profile in `~/.config/bananahub/config.json`
  3. Environment variables fill missing fields by default (`OPENAI_API_KEY`, `OPENAI_BASE_URL`, `GOOGLE_API_KEY`, `GEMINI_API_KEY`, `BANANAHUB_PROVIDER`, `BANANAHUB_AUTH_MODE`, `BANANAHUB_MODEL`, `GOOGLE_GEMINI_BASE_URL`, `GEMINI_BASE_URL`, `BANANAHUB_BASE_URL`, `GOOGLE_CLOUD_PROJECT`, `GOOGLE_CLOUD_LOCATION`)
     - Use `BANANAHUB_PROFILE=<name>` to select another persisted profile
     - Set `BANANAHUB_ENV_OVERRIDE=1` only when environment variables should temporarily override profile fields
  4. Skill config examples:
     - `{"provider": "google-ai-studio", "api_key": "...", "model": "gemini-3-pro-image-preview"}`
     - `{"provider": "gemini-compatible", "api_key": "...", "base_url": "https://..."}`
     - `{"provider": "openai", "openai_api_key": "...", "model": "gpt-image-2"}`
     - `{"provider": "openai-compatible", "openai_api_key": "...", "openai_base_url": "https://...", "model": "gpt-image-2"}`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bananahub-ai/bananahub-skill](https://github.com/bananahub-ai/bananahub-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

---
trigger: always_on
description: **This file provides context and instructions for AI coding agents and assistants working in this workspace.**
---

# AGENTS.md

**This file provides context and instructions for AI coding agents and assistants working in this workspace.**

**Version:** June 2026 (Updated for Grok Imagine Cinematic Studio v3.6.5, Grok plugin marketplace, model stack, and AI Polish Director)  
**Canonical Source:** https://github.com/FineComputer14451/Grok-Imagine-Cinematic-Studio/blob/main/AGENTS.md

Think of this as the single source of truth for how to interact with this Grok/xAI agent environment in `/home/workdir/`.

## Workspace Overview

This is a persistent Linux sandbox environment (`/home/workdir/`) designed for advanced Grok agent workflows, with heavy emphasis on:

- Custom skill development and orchestration
- High-quality cinematic image/video generation pipelines (Grok Imagine)
- Document, presentation, and media production
- GitHub repository management and open-source contribution
- Animal welfare legal research & advocacy tooling (supporting user's ongoing work)

**Core principle:** Use the appropriate skill or tool for every task. Do not reinvent wheels that skills already handle. Prefer existing skills over ad-hoc scripts.

## Directory Structure

```
/home/workdir/
├── .grok/
│   └── skills/                  # All custom skills live here (one per subdirectory)
│       ├── <skill-name>/
│       │   ├── SKILL.md         # Required: YAML frontmatter + imperative instructions
│       │   ├── scripts/         # Optional: executable helpers
│       │   ├── references/      # Optional: long-form docs, production bibles, agent defs
│       │   └── assets/          # Optional: templates, reference images, etc.
├── .grok-plugin/                # Grok plugin manifests (marketplace.json, plugin.json, plugin-index.json for 44 skills + commands)
├── artifacts/                   # All outputs go here (images, docs, videos, code, etc.)
├── scripts/                     # Install/verify/update helpers + generate_plugin_index.py
├── web_ui/                      # Streamlit dashboard (model pickers, quota sim, DNA/sequence tools)
├── AGENTS.md                    # This file (you are here)
├── README.md                    # Human-facing overview (keep in sync)
├── CHANGELOG.md
├── RELEASE_NOTES_v3.6.md
├── Quick_Start_Guide.md
└── (other project files as added: tools/, references/agents/, examples/, commands/, etc.)
```

## Skill System Rules (Critical)

When working with or creating skills:

1. **Always follow the official skill-creator guidelines** first — read `/root/.grok/skills/skill-creator/SKILL.md`.
2. Every skill **must** have a `SKILL.md` with strict YAML frontmatter:
   - `name`: kebab-case, matches directory name exactly
   - `description`: single-line plain text (no colons, no `<`/`>`, max 1024 chars) describing **when to use** this skill
3. **Never** create `README.md`, `CHANGELOG.md`, or human-facing docs inside skill directories — skills are agent-only.
4. Keep `SKILL.md` concise (< ~500 lines). Move detailed content, agent personalities, production bibles, and long references to `references/`.
5. New skills **must** be created in `/home/workdir/.grok/skills/<name>/` using the init script from skill-creator.
6. Validate after creation: `bash /root/.grok/skills/skill-creator/scripts/validate-skill.sh <skill-dir>`

## Common Workflows & Commands

### File Operations
- Read: `read_file` (supports `offset` + `limit`)
- Write/Edit: `write_file`, `edit_file`
- Explore: `bash ls -la`, `bash find`, `bash tree`

### Image & Media Tasks (Grok Imagine)
- **Generate new images**: `generate_image` (detailed prompt + orientation)
- **Edit existing / generated images**: `edit_image` (prompt + `file_path` or `image_id`)
- **AI-powered recreation / style transfer / enhancement** of uploaded images: Activate `ai-image-recreation`
- **Extract Character DNA** for consistency: Activate `character-dna-extractor`
- **Extend cinematic sequences** (60–120s+): Activate `cinematic-sequence-extender` or `extend-frame-to-video`
- **Refine / iterate on previously generated images**: `generated-image-editor`
- **Upscale video for final delivery** (720p → 1080p/4K, face restoration): Activate `ai-video-upscaler`
- Video / audio processing: Activate `ffmpeg` skill or use bash directly
- **Full cinematic production**: Activate `grok-imagine-cinematic-studio` (23-agent + specialist suite, v3.6.5 with plugin support)

### Document Tasks
- PDF: `pdf` skill
- Word (.docx): `docx` skill
- PowerPoint (.pptx): `pptx` skill
- Excel (.xlsx): `xlsx` skill

### GitHub & Connected Services
- All GitHub operations: Activate `github-repo-manager` skill first
- Discover connected services (GitHub, Gmail, Outlook, Google Drive, Canva): `search_connected_tools`
- Then execute with `call_connected_tool`

### Grok Plugins & Marketplace
- Install/update the full Cinematic Studio: `grok plugin install FineComputer14451/Grok-Imagine-Cinematic-Studio --trust`
- Or via marketplace: `grok plugin marketplace add FineComputer14451/Grok-Imagine-Cinematic-Studio` then install by name
- Regenerate index after skill changes: `python scripts/generate_plugin_index.py`
- Validate plugin: `grok plugin validate` + check `.grok-plugin/plugin-index.json`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FineComputer14451/Grok-Imagine-Cinematic-Studio](https://github.com/FineComputer14451/Grok-Imagine-Cinematic-Studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->

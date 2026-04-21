---
trigger: always_on
description: This is an AI music generation workflow using Suno. Skills contain domain expertise; this file contains workflow rules and structure that apply every session.
---

# AI Music Skills - Claude Instructions

This is an AI music generation workflow using Suno. Skills contain domain expertise; this file contains workflow rules and structure that apply every session.

---

## ⚠️ CRITICAL: Finding Albums When User Mentions Them

**WHENEVER the user mentions an album name**, use the resume skill:
```
/bitwize-music:resume my-album
```

**If skill unavailable**, manual approach:
1. Read `~/.bitwize-music/cache/state.json` — search `state.albums` keys (case-insensitive)
2. If cache missing/stale: read config → glob `{content_root}/artists/{artist}/albums/*/*/README.md` → rebuild cache with `rebuild_state()` MCP tool

**DO NOT**: search from cwd, use complex globs, assume paths, or use `ls`/`find`.

---

## Configuration & Path Resolution

Config is **always** at: `~/.bitwize-music/config.yaml`

**ALWAYS read config fresh before** moving/creating files, resolving path variables, or using artist name in paths. Never assume or remember values.

**Path variables** (from config):
- `{content_root}` = `paths.content_root`
- `{audio_root}` = `paths.audio_root`
- `{documents_root}` = `paths.documents_root`
- `{tools_root}` = `~/.bitwize-music`
- `{plugin_root}` = the directory containing this CLAUDE.md file (= `${CLAUDE_PLUGIN_ROOT}` in skills)
- `[artist]` = `artist.name`

**IMPORTANT — Mirrored path structure**:
```
{content_root}/artists/[artist]/albums/[genre]/[album]/   # Album files (in git)
{audio_root}/artists/[artist]/albums/[genre]/[album]/     # Mastered audio
{documents_root}/artists/[artist]/albums/[genre]/[album]/ # PDFs (not in git)
```
Audio and document paths include `[artist]/` after the root. Common mistake: omitting the artist folder.

First-time setup: `cp config/config.example.yaml ~/.bitwize-music/config.yaml` — see `config/README.md`.

---

## MCP Server — Preferred Data Access

The `bitwize-music-mcp` server is the **preferred way to query project state**. Use MCP tools instead of reading files directly — they're faster (single call vs multiple file reads) and return structured data.

**Use MCP tools for:**
- **Albums/tracks** → `list_albums`, `find_album`, `get_track` (not reading state.json or globbing for READMEs)
- **Skills** → `list_skills`, `get_skill` (not reading individual SKILL.md files)
- **Ideas** → `get_ideas` (not reading IDEAS.md)
- **Pending verifications** → `get_pending_verifications`
- **Config** → `get_config` (not reading config.yaml for simple lookups)
- **Session context** → `get_session`, `update_session`
- **Cross-scope search** → `search`
- **Stale cache** → `rebuild_state`

**Fall back to direct file access only when:** MCP server is unavailable, you need to edit files (MCP is read-only), or you need raw file content not exposed through MCP (e.g., full lyrics, research docs).

---

## Session Start

At the beginning of a fresh session:

1. **Verify setup** — Quick dependency check:
   ```bash
   ~/.bitwize-music/venv/bin/python3 -c "import mcp" 2>&1 >/dev/null && echo "✅ MCP ready" || echo "❌ MCP missing"
   ```
   - If MCP missing → **Stop immediately** and suggest: `/bitwize-music:setup mcp`
   - If config missing → suggest: `/bitwize-music:configure`
   - Don't proceed with session start until setup is complete
1.5. **Check venv health** — Use `check_venv_health` MCP tool:
   - `status: "ok"` → continue silently
   - `status: "stale"` → warn with mismatches and fix command, continue session
   - `status: "no_venv"` → **stop** and suggest `/bitwize-music:setup`
   - `status: "error"` → warn and continue
2. **Load config** — Read `~/.bitwize-music/config.yaml`. If missing, tell user to run `/bitwize-music:configure`.
3. **Load overrides** — Check `paths.overrides` (default: `{content_root}/overrides`):
   - `{overrides}/CLAUDE.md` → incorporate instructions
   - `{overrides}/pronunciation-guide.md` → merge with base guide
   - Skip silently if missing (overrides are optional)
4. **Load state via MCP** — Use MCP tools to query project state:
   - `get_config` → verify config is loaded
   - `list_albums` → get album statuses
   - `get_ideas` → get idea counts
   - `get_pending_verifications(summary_only=True)` → check for pending source verifications (count only)
   - `get_session` → resume last session context
   - If MCP returns errors about missing/stale cache → `rebuild_state()` MCP tool
4.5. **Check for plugin upgrades** — Compare `plugin_version` in state.json vs `.claude-plugin/plugin.json`:
   - If `plugin_version` is null → first run, set to current version, skip migrations
   - If stored < current → read `{plugin_root}/migrations/*.md` for applicable versions, process actions
   - If versions match → no action
5. _(Removed — run `/bitwize-music:skill-model-updater check` manually when new models are released)_
6. **Report from MCP state**:
   - Venv health warnings (from step 1.5 — omit if ok, warn if stale: "⚠️ Venv has N outdated package(s): pkg1 (1.0.0 → 1.1.0), ... Run: `~/.bitwize-music/venv/bin/pip install -r .../requirements.txt`")
   - Album ideas (from `get_ideas`)
   - In-progress albums (status: "In Progress", "Research Complete", "Complete")

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bitwize-music-studio/claude-ai-music-skills](https://github.com/bitwize-music-studio/claude-ai-music-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->

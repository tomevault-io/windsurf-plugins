---
trigger: always_on
description: > If this checkout lives inside a larger workspace, check the parent directory (`../CLAUDE.md` / `../AGENTS.md`) for additional agent guidance — some agents don't walk up on their own.
---

> If this checkout lives inside a larger workspace, check the parent directory (`../CLAUDE.md` / `../AGENTS.md`) for additional agent guidance — some agents don't walk up on their own.

## Development notes

- ALWAYS USE TAILWINDCSS. No custom CSS unless absolutely necessary.
- **This is a uv project.** Always use `uv run` to execute Python commands in the backend (e.g., `uv run python`, `uv run pytest`). Never try to activate the venv manually.
- This project has a comprehensive CLI at tools/stimma which abstracts devex tasks. See below for help info.
- Do not start or restart development servers automatically. Assume the developer manages long-running frontend and backend processes; ask before restarting them.
- Always implement soft-delete for new database tables.
- Use trash icons for delete actions, not X icons - this is the project convention.
- **Built-in stimpack SOURCE lives in a sibling repo, not here:** `../stimma-skills` (i.e. `~/stimma/stimma-skills`), one directory per stimpack. They are *not* loaded from this repo — at runtime the app loads stimpacks from the profile dir (after cloud marketplace install). For dev, point the app at the sibling repo with `stimma stimpacks dev` (writes `dev_stimpacks_dir` to config.yaml); those stimpacks then shadow the profile-installed copies live, so editing `../stimma-skills` needs no publish/install. See backend `agent/v2/stimpacks.py` (`_dev_stimpacks_dir` / `_iter_effective_stimpack_dirs`).

## Database Guidelines

Database migrations are handled using alembi and run automatically at startup. Do not modify the database schema any other way. 

## Data Directories

Stimma uses a two-level directory scheme: **bundle ID** (release channel) + **sandbox** (isolated instance).

```
~/Library/Application Support/<bundle-id>/<sandbox>/
```

Bundle IDs: `ai.stimma.stimma` (stable), `ai.stimma.stimma.debug` (dev, default for CLI).

The CLI defaults to the `debug` channel and `default` sandbox. Use `--channel=X` or `--prod` to switch channels, `--sandbox=NAME` for sandboxes.

See `docs/DATA_DIRECTORIES.md` for full details.

Config files are located at:
- `~/Library/Application Support/ai.stimma.stimma/default/config.yaml` (PRODUCTION)
- `~/Library/Application Support/ai.stimma.stimma.debug/default/config.yaml` (DEVELOPMENT)


## UI Guidelines

- When building code that adds a new entity, don't pop up a modal asking the person to name it. Just create it with no name or Untitled or whatever makes sense and make it easy for the user to rename later.
- Never use browser based "alert" sheets for confirmations or prompts to the user. Use modals instead.

### Design language — READ frontend/DESIGN.md FIRST

All UI work follows the Atelier v3 design language, specified in
`frontend/DESIGN.md` (tokens, z-scale, radius roles, row grammars, separator
law, greppable review rules). Read it before building or restyling anything.
Quick anchors: teal `accent` = actions; indigo `selection` = selected state;
magenta `live` = user-armed continuous modes; `matte` behind media;
`rounded-media` (2px) on artwork; sentence-case section labels with no rule
under headings; hairlines only BETWEEN peers; blue-500 is status-only (never
an interactive accent). Use the kit in `frontend/src/components/ui/` instead
of hand-rolling controls.

### Stimma Cloud Branding

**All Stimma Cloud UI elements must use the Stimma Cloud gradient branding.** The GRADIENT (and gradient text/borders) is cloud-reserved — never on non-cloud features. Solid teal via the `accent` token is the app-wide interactive accent and is fine everywhere; solid indigo via `selection` likewise. What is reserved is the teal→cyan→indigo gradient treatment, not the individual hues.

**Core Gradient:** `teal-600` → `cyan-500` → `indigo-500`
- CSS variables: `--stimma-cloud-start: #0d9488`, `--stimma-cloud-mid: #06b6d4`, `--stimma-cloud-end: #6366f1`
- Tailwind: `from-teal-600 via-cyan-500 to-indigo-500`

**CSS Utility Classes** (defined in `style.css`):
- `.stimma-cloud-text` - Gradient text effect
- `.stimma-cloud-border` - Animated gradient border (40% opacity, 60% on hover)
- `.stimma-cloud-border-solid` - Solid gradient border (60% opacity)

**Common Patterns:**
- Gradient borders: Wrap with `p-[1px] bg-gradient-to-r from-teal-600/40 via-cyan-500/40 to-indigo-500/40`, inner `bg-zinc-900/90`
- CTA buttons: `bg-gradient-to-r from-teal-600 via-cyan-500 to-indigo-500 text-white`
- Hover: `hover:from-teal-500 hover:via-cyan-400 hover:to-indigo-400`

**Tier Badge Colors:**
| Tier | Background | Text |
|------|-----------|------|
| Power | `bg-indigo-900/50` | `text-indigo-400` |
| Pro | `bg-cyan-900/50` | `text-cyan-400` |
| Free (BYOAI) | `bg-zinc-700/50` | `text-zinc-400` |

## Important Docs

See the docs/ folder. In particular:

- If you are working on tools or the Stimma Tools Protocol (STP), see the spec at https://github.com/stimma-ai/stimma-tools-protocol
- If you are working on local Stimma Cloud target overrides, see docs/CLOUD_TARGETS.md. Keep private staging hostnames and Access tokens out of this OSS repo; use ignored `.env.local` values instead.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stimma-ai/stimma](https://github.com/stimma-ai/stimma) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->

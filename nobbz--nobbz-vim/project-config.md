---
trigger: always_on
description: **Copilot is restricted to read-only access on this repository.**
---

# Copilot Onboarding Instructions for `nobbz-vim`

## Copilot Permissions

**Copilot is restricted to read-only access on this repository.**

Copilot MAY:
- Read any file in the repository
- Discuss, review, and analyse the codebase
- Suggest changes and explain their rationale

Copilot MAY NOT:
- Modify, create, or delete any project files

The only files Copilot is permitted to write are agent/LLM-specific instruction files:
- `AGENTS.md`
- `CLAUDE.md` (or any equivalent agent-specific instruction file)
- `.github/copilot-instructions.md` (this file)
- Anything under `.opencode/`

## Repository Overview

This is a Neovim configuration managed as a Nix flake. The repository provides a reproducible, declarative Neovim setup with custom plugins and LSP configurations.

**Key Facts:**
- **Purpose:** Personal Neovim configuration using Nix flakes for reproducibility
- **Size:** Small (~600KB, ~1900 lines of Nix/Lua code)
- **Languages:** Nix, Lua, Python (for build scripts)
- **Target Users:** Developers in the Nix ecosystem
- **Try it:** `nix run github:nobbz/nobbz-vim` (requires Nix with flakes enabled)

## Build and Development Commands

> **Reference only.** Copilot must not run these commands or modify project files.

**Prerequisites:** Nix with flakes enabled. This project REQUIRES Nix - all build/test commands use Nix.

### Core Commands (all require Nix)

1. **Build the package:** `nix build` or `nix build .#nobbzvim`
   - Builds the Neovim configuration with all plugins
   - Output in `./result` symlink
   - Takes 1-2 minutes on first build (downloads dependencies)

2. **Run directly:** `nix run` or `nix run .#nobbzvim`
   - Launches Neovim with the configuration
   - Useful for quick testing

3. **Enter dev shell:** `nix develop`
   - Provides: `nil`, `stylua`, `npins`, `alejandra`, `basedpyright`, `emmy-lua-code-style`
   - Creates `.luarc.json` symlink for LSP support
   - Use this for making changes to Lua code

4. **Format code:** `nix fmt`
   - Formats all Nix files with `alejandra`
   - Formats all Lua files with `emmy-lua-code-style` (CodeFormat)
   - **ALWAYS run before committing** - formatting is strict
   - Respects `.editorconfig`
   - Note: `stylua` is available in the dev shell as a standalone tool but is **not** part of `nix fmt`

5. **Update flake lock:** `nix flake update --commit-lock-file`
   - Updates all flake inputs (nixpkgs, etc.)
   - Automatically commits the lock file

6. **Update plugins:** `nix run .#update-plugins`
   - Updates all nvim-* plugins tracked in `npins/sources.json`
   - Commits each plugin update separately
   - May take several minutes

7. **Add new plugin:** `nix run .#add-plugin <name> <owner/repo>`
   - Example: `nix run .#add-plugin telescope nvim-telescope/telescope.nvim`
   - Adds plugin to `npins/sources.json` with `nvim-` prefix
   - Optionally use `-b branch` or `-t gitlab` flags

8. **Run checks:** `nix flake check`
   - Validates flake structure
   - Checks `add-plugin` and `update-plugins` packages build successfully

### Common Workflows

> **Reference only.** Copilot must not perform these steps.

**Making changes to Lua configuration:**
1. `nix develop` - enter dev shell
2. Edit files in `plugins/nobbz/lua/nobbz/`
3. Test with `:checkhealth nobbz` in Neovim
4. `nix fmt` - format before committing

**Adding a new plugin:**
1. `nix run .#add-plugin <name> <owner/repo>`
2. Edit `nix/mnw/default.nix` - add plugin to `plugins.start` list (mandatory) or `plugins.opt` list (optional)
3. Configure in `plugins/nobbz/lua/nobbz/` - create new file or edit existing
4. Add lazy loading spec if optional plugin
5. `nix build` - verify it builds
6. `nix fmt` - format before committing

## Project Structure

### Root Files
- `flake.nix` - Main flake definition, defines packages and dev shell
- `flake.lock` - Lock file for flake inputs (auto-generated, commit changes)
- `nix/mnw/default.nix` - MNW (Minimal Neovim Wrapper) configuration for Neovim package
- `nix/mnw.nix` - MNW integration module, defines `nobbzvim` and `nobbzvide` packages
- `.editorconfig` - Lua formatting rules (2 spaces, double quotes, comma separators)
- `.stylua.toml` - Stylua configuration (mostly defaults explicitly set)
- `.envrc` - direnv configuration for automatic dev shell loading
- `.gitignore` - Ignores `.luarc.json` (generated) and `.direnv`

### Directories

**`plugins/`** - Plugin management
- `default.nix` - Flake-parts module, builds vim plugins from npins entries with `nvim-` prefix
- `nobbz/` - Custom plugin containing all configuration
  - `lua/nobbz/` - Lua configuration modules
  - `plugin/nobbz.lua` - Plugin entry point

**`plugins/nobbz/lua/nobbz/`** - Configuration modules (one file per feature)
- `init.lua` - Main entry point, loads all submodules
- `lazy/` - Custom lazy-loading system wrapping `lz.n` (`init.lua`, `specs.lua`)
- `lsp/` - LSP server configurations (one file per language)
- `health.lua` - Custom health check system
- Feature-specific files: `blink.lua`, `telescope.lua`, `lualine.lua`, etc.

**`nix/`** - Nix modules
- `mnw.nix` - Defines `nobbzvim` and `nobbzvide` packages via MNW and wrapper-manager
- `mnw/default.nix` - MNW configuration: plugin lists (`plugins.start`, `plugins.opt`), LSP binaries via `extraBinPath`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NobbZ/nobbz-vim](https://github.com/NobbZ/nobbz-vim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-11 -->

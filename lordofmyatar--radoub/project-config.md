---
trigger: always_on
description: Project guidance for Claude Code sessions working with the Radoub multi-tool repository.
---

# CLAUDE.md - Radoub Toolset

Project guidance for Claude Code sessions working with the Radoub multi-tool repository.

---

## Repository Overview

**Radoub** is a multi-tool repository for Neverwinter Nights modding. Each tool maintains its own codebase, documentation, and development workflow within its subdirectory.

### Current Tools

| Tool | Abbrev | Description | CLAUDE.md |
|------|--------|-------------|-----------|
| **Parley** | PAR | Dialog editor (`.dlg` files) | `Parley/CLAUDE.md` |
| **Manifest** | MAN | Journal editor (`.jrl` files) | `Manifest/CLAUDE.md` |
| **Quartermaster** | QM | Creature/inventory editor (`.utc`, `.bic` files) | `Quartermaster/CLAUDE.md` |
| **Fence** | FEN | Merchant/store editor (`.utm` files) | `Fence/CLAUDE.md` |
| **Relique** | REL | Item blueprint editor (`.uti` files) | `Relique/CLAUDE.md` |
| **Reliquary** | RLQ | Placeable blueprint editor (`.utp` files; namespace `PlaceableEditor`) | `Reliquary/CLAUDE.md` |
| **Trebuchet** | TRE | Radoub launcher/hub | `Trebuchet/CLAUDE.md` |
| **Marlinspike** | MAR | Search & replace across files (lives in Trebuchet) | `Trebuchet/CLAUDE.md` |
| **Radoub** | RAD | Repository-level / shared | This file |

### Shared Libraries

- **Radoub.Formats**: Aurora Engine file format parsers (KEY, BIF, etc.) - Shared library for all tools

### Planned Tools

None currently. Future tools land as subdirectories with their own README, CLAUDE.md, and development infrastructure. Bootstrap follows the [New Tool Bootstrap guide](Documentation/NEW_TOOL_BOOTSTRAP.md).

---

## Repository Structure

```
Radoub/
├── README.md (landing page)
├── LICENSE
├── CLAUDE.md (this file - repo-level guidance)
├── Radoub.sln (builds all tools; excludes Windows-only integration tests)
├── .gitignore
├── .claude/commands/ (slash commands for Claude Code)
├── Documentation/ (Aurora Engine format specs - shared across tools)
│   └── BioWare_Original_PDFs/ (original BioWare PDFs)
├── About/ (project history and AI collaboration documentation)
│   ├── CLAUDE_DEVELOPMENT_TIMELINE.md
│   └── ON_USING_CLAUDE.md
├── NonPublic/ (private docs, specs, research — NOT in git)
│   ├── Relique/ (Relique specs, plans, research)
│   ├── Parley/ (Parley specs, plans, research)
│   ├── Quartermaster/ (QM specs, plans, research)
│   ├── Fence/ (Fence assets, research)
│   └── Trebuchet/ (Trebuchet research)
├── Parley/ (dialog editor)
│   ├── README.md
│   ├── CLAUDE.md (Parley-specific guidance)
│   ├── CHANGELOG.md (Parley-specific changes)
│   ├── Parley/ (source code)
│   ├── TestingTools/
│   ├── Documentation/ (Approved Parley-specific docs)
├── Manifest/ (journal editor)
│   ├── CLAUDE.md (Manifest-specific guidance)
│   ├── CHANGELOG.md (Manifest-specific changes)
│   ├── Manifest/ (source code)
│   └── Manifest.Tests/ (unit tests)
├── Quartermaster/ (creature/inventory editor)
│   ├── CLAUDE.md (Quartermaster-specific guidance)
│   ├── CHANGELOG.md (Quartermaster-specific changes)
│   ├── Quartermaster/ (source code)
│   └── Quartermaster.Tests/ (unit tests)
├── Fence/ (merchant/store editor)
│   ├── CLAUDE.md (Fence-specific guidance)
│   ├── CHANGELOG.md (Fence-specific changes)
│   ├── Fence/ (source code)
│   └── Fence.Tests/ (unit tests)
├── Relique/ (item blueprint editor)
│   ├── CLAUDE.md (Relique-specific guidance)
│   ├── CHANGELOG.md (Relique-specific changes)
│   ├── Relique/ (source code, namespace: ItemEditor)
│   └── Relique.Tests/ (unit tests)
├── Trebuchet/ (launcher/hub)
│   ├── CLAUDE.md (Trebuchet-specific guidance)
│   ├── CHANGELOG.md (Trebuchet-specific changes)
│   ├── Trebuchet/ (source code)
│   └── Trebuchet.Tests/ (unit tests)
├── Radoub.Formats/ (shared library)
│   ├── Radoub.Formats.sln
│   ├── Radoub.Formats/ (source code)
│   └── Radoub.Formats.Tests/ (unit tests)
└── [Future tools will be added here]
```

---

## Working with Multiple Tools

### Building

**Root-level solution**: Use `Radoub.sln` to build all tools at once:

```bash
# Build all tools (excludes Windows-only integration tests)
dotnet build Radoub.sln

# Build with release configuration
dotnet build Radoub.sln --configuration Release
```

**Individual tool builds**:
```bash
dotnet build Parley/Parley.sln
dotnet build Manifest/Manifest/Manifest.csproj
dotnet build Quartermaster/Quartermaster/Quartermaster.csproj
```

**Note**: `Radoub.IntegrationTests` is excluded from `Radoub.sln` because it targets `net9.0-windows` (FlaUI requires Windows).

### Tool-Specific Work

When working on a specific tool (e.g., Parley):
1. **Always read the tool's CLAUDE.md first** (`Parley/CLAUDE.md`)
2. Follow tool-specific conventions and workflows
3. Tool-specific issues/PRs reference the tool in title: `[Parley] Fix parser bug`
4. Run tool-specific tests before committing

### Shared Resources

**Public Documentation** (`Documentation/`):
- Shared across all tools
- Contains original BioWare PDF format specifications
- Read-only reference material
- Markdown conversions of BioWare docs are now in the Wiki (see Resources section)
- **NEVER read PDF files without explicit user instruction** - PDFs can exceed context limits and cause "prompt too long" errors

**About Documentation** (`About/`):
- Project history and development experience

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LordOfMyatar/Radoub](https://github.com/LordOfMyatar/Radoub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->

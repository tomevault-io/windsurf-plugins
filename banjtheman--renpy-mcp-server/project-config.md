---
trigger: always_on
description: Generates images via Gemini API:
---

# CLAUDE.md - RenPy Studio Development Guide

## Project Overview

RenPy Studio is a TypeScript MCP App that provides a visual studio interface for creating Ren'Py visual novels. It's published as `renpy-studio` on npm and can be installed via `npx -y renpy-studio --stdio`.

**Repository**: https://github.com/banjtheman/renpy_mcp_server (in `renpy_mcp_app/` folder)

## Architecture

```
renpy_mcp_app/
├── src/
│   ├── index.ts              # Main entry, registers view_studio tool
│   ├── main.ts               # Server startup (stdio/http modes)
│   ├── provision/            # Project workspace & SDK management
│   │   ├── index.ts          # WORKSPACE_DIR constant (~/.renpy-studio/workspace)
│   │   ├── renpy-sdk.ts      # Auto-downloads Ren'Py SDK
│   │   └── venv.ts           # Python virtual environment setup
│   ├── server/
│   │   ├── tools/
│   │   │   ├── project-tools.ts   # create_project, list_projects, delete_project
│   │   │   ├── asset-tools.ts     # generate_character/background, ui_get_assets, ui_delete_asset
│   │   │   ├── script-tools.ts    # generate_script, get_script_graph, read/edit_project_file
│   │   │   └── build-tools.ts     # build_project, start/stop_web_preview
│   │   └── python-runner.ts  # Executes Python scripts via venv
│   ├── ui/
│   │   ├── studio.tsx        # Main React UI component
│   │   ├── styles.css        # All styling
│   │   └── components/
│   │       └── storybuilder/
│   │           ├── StoryBuilderView.tsx     # Visual story flow component
│   │           ├── BeatCard.tsx             # Individual story beat cards
│   │           └── visual-builder-types.ts  # Beat expansion algorithm
│   └── lib/
│       ├── renpy-parser.ts   # Parses Ren'Py scripts → story graph
│       └── story-types.ts    # SceneNode, DialogueLine, MenuChoice, etc.
├── python/
│   ├── image_service.py      # Gemini image generation
│   ├── background_remover.py # rembg background removal + 750px normalization
│   ├── build_manager.py      # Ren'Py web build
│   └── preview_manager.py    # Local preview server
├── bin/
│   └── cli.js                # CLI entry point for npx
└── dist/                     # Built output (server + bundled UI)
```

## Key Concepts

### Project Structure

Projects are stored in `~/.renpy-studio/workspace/` with this structure:
```
{project_name}/
  game/
    script.rpy      # Main script (has label start)
    *.rpy           # Additional scripts
    images/         # Copied from assets during build
  assets/
    background/     # Generated background images
    character/      # Character sprites (*_transparent.png)
    metadata.json   # Asset descriptions and prompts for regeneration
```

### Asset Metadata System

Each project has `assets/metadata.json` storing generation prompts:
```json
{
  "version": 1,
  "characters": {
    "alice": {
      "description": "friendly barista with brown hair",
      "role": "Main character",
      "displayName": "Alice"
    }
  },
  "backgrounds": {
    "cafe_interior": {
      "description": "Cozy cafe with warm lighting",
      "location": "Indoor",
      "timeOfDay": "Afternoon"
    }
  }
}
```

This enables:
- Displaying descriptions in the Asset Gallery
- "Regenerate" button using original prompts
- Persistence across sessions

### Character Image Pipeline

1. `generate_character` calls `image_service.py` to generate images via Gemini
2. Images saved to `assets/character/` as `{name}_{emotion}.png`
3. `background_remover.py` removes background and normalizes to 750px height
4. Saved as `{name}_{emotion}_transparent.png`
5. Metadata saved to `metadata.json`

### Visual Story Builder

The story builder parses Ren'Py scripts and displays story flow:

1. **Parser** (`renpy-parser.ts`):
   - Extracts labels, dialogue, menus, visual events
   - Tracks multiple menus per label with `SceneMenu[]`
   - Captures inline choice branches with `inlineDialogue`

2. **Beat Expansion** (`visual-builder-types.ts`):
   - Converts scenes to individual "beats" (dialogue moments)
   - Creates branch/merge connections for choices
   - Tracks visual state (background, characters) at each beat

3. **BeatCard** (`BeatCard.tsx`):
   - Shows mini preview with background and up to 3 characters
   - Displays dialogue text and speaker
   - Indicates choice branches with gold styling

### Script Parser Types

```typescript
interface SceneMenu {
  atDialogueIndex: number;  // Where in dialogue[] this menu appears
  prompt?: string;          // "How do you respond?"
  choices: MenuChoice[];
}

interface MenuChoice {
  text: string;
  target: string;           // Label to jump to (empty if inline)
  inlineDialogue?: DialogueLine[];
  inlineVisualEvents?: VisualEvent[];
}
```

## MCP Tools

### Project Management
- `create_project` - Creates new project with directory structure
- `list_projects` - Lists all projects with metadata
- `delete_project` - Removes a project and all files

### Asset Generation
- `generate_character` - Creates character sprites with 5 emotions
- `generate_background` - Creates scene backgrounds
- `ui_get_assets` - Returns all assets with base64 data and metadata (app-only)
- `ui_delete_asset` - Deletes an asset and its metadata (app-only)

### Script Management
- `generate_script` - Creates new .rpy file with auto-validation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [banjtheman/renpy_mcp_server](https://github.com/banjtheman/renpy_mcp_server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->

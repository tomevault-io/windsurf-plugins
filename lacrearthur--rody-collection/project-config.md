---
trigger: always_on
description: **Always invoke the `developing-unity-games` skill when working in this repo.**
---

# CLAUDE.md

**Always invoke the `developing-unity-games` skill when working in this repo.**

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Rody Collection** is a Unity 6 (6000.3.2f1) recreation of classic Atari ST point-and-click adventure games with a built-in story editor. The game supports dual platforms: desktop (local storage) and WebGL (embedded Resources).

**Live URL**: https://lacrearthur.github.io/rody-collection/

## Quick Reference

| Key | Value |
|-----|-------|
| Unity Version | 6000.3.2f1 (Unity 6 LTS) |
| Render Pipeline | Universal Render Pipeline (URP) |
| Input System | Both legacy and new |
| Target Resolution | 960x600 (3x Atari ST 320x200) |

## Build Commands

```bash
# Build WebGL from Unity Editor
# File > Build Settings > WebGL > Build
# Output: build/WebGL/

# Build WebGL from CLI (used by CI)
# Unity -batchmode -quit -projectPath . -buildTarget WebGL -executeMethod BuildScript.Build

# Deploy to GitHub Pages (CI builds on push to master)
git push origin master  # Triggers .github/workflows/deploy-pages.yml
```

**⚠️ CI Note**: Do NOT push to `master` on every commit. CI builds WebGL on each push to master. Batch commits locally and push when ready for a build.

## Editor Tools

| Menu Path | Purpose |
|-----------|---------|
| `Tools > Rody > Export Stories to JSON` | Opens batch export window |
| `Tools > Rody > Export All Stories Now` | Export from `./original-stories/` to `Resources/Stories/` |

## Local-Only Plugins

These plugins are installed locally but excluded from git (add them manually):
- **Odin Inspector** - Enhanced Unity inspector

Note: DOTween is now included in the repo for CI builds.

## Architecture

### Story Storage (Unified)
| Type | Location | Access |
|------|----------|--------|
| Official Stories | `Resources/Stories/*.rody.json` | Read-only, embedded in build |
| User Stories | User's file system | Import/export via file picker (no persistent storage) |

- **Abstraction**: `IStoryProvider` interface in `Assets/Scripts/Providers/`
- **JSON format**: `.rody.json` files with base64-encoded sprites
- **Source stories**: `./original-stories/` (folder-based, for re-export only via Editor tools)

### User Stories Feature
User-created content uses import/export model (no persistent storage):
- **Fork-on-Edit**: Editing official story creates in-memory copy via `WorkingStory.ForkForEditing()`
- **Export/Import**: Portable `.rody.json` format with base64 sprites
- **Key file**: `WorkingStory.cs` - single in-memory story state

**WorkingStory API** (key methods):
```csharp
// Loading
WorkingStory.LoadOfficial(storyId)      // Load from Resources
WorkingStory.LoadFromJson(json, path)   // Import user story
WorkingStory.CreateNew(title)           // Blank story

// Editing
WorkingStory.ForkForEditing()           // Copy official for editing
WorkingStory.SaveScene(index, data)     // Update scene
WorkingStory.CreateNewScene(index)      // Add scene

// Export
WorkingStory.ExportToJson()             // Get JSON string
WorkingStory.MarkSaved(path)            // Clear dirty flag

// State
WorkingStory.IsLoaded, .IsOfficial, .IsDirty, .Title, .SceneCount
```

### Scene Build Order
| Index | Scene | Purpose |
|-------|-------|---------|
| 0 | 0_MenuCollection | Story selection |
| 1 | 1_TitleScene | Story title card |
| 2 | 2_MenuScene | In-game menu |
| 3 | 3_GameScene | Main gameplay |
| 4 | 4_CreditsScene | Story credits |
| 5 | 5_WinScene | Completion screen |
| 6 | RM_Main | Level Editor |

### Initialization Flow
```
Bootstrap.cs → StoryProviderManager.Initialize() → Provider ready
    ↓
Scene 0 loads stories via Provider → User selects story → Scenes 1-5 for gameplay
    ↓
Scene 6 for editing (accessible via Edit button)
```

### Key Code Paths
| Path | Purpose |
|------|---------|
| `Assets/Scripts/GameManager.cs` | Main gameplay controller |
| `Assets/Scripts/RodyMaker/` | Level editor (RM_ prefix) |
| `Assets/Scripts/RodyAnthology/` | Story selection (RA_ prefix) |
| `Assets/Scripts/Providers/WorkingStory.cs` | In-memory story state (THE source of truth) |
| `Assets/Scripts/Providers/ResourcesStoryProvider.cs` | Loads official stories from Resources |
| `Assets/Scripts/Providers/StoryProviderManager.cs` | Singleton provider access |
| `Assets/Scripts/Models/SceneData.cs` | Typed scene data model |
| `Assets/Scripts/Utils/PathManager.cs` | Cross-platform path handling |
| `Assets/Scripts/WebGL/WebGLFileBrowser.cs` | WebGL file import/export via browser |
| `Assets/Scripts/SoundManager.cs` | Phoneme TTS and audio |
| `Assets/Editor/StoryExportTool.cs` | Batch export stories to JSON |

### Phoneme Text-to-Speech
Custom TTS concatenates pre-recorded phonemes. Syntax: `b_r_a_v_o` (underscores between phonemes, spaces between words).
- Phoneme indices: `P.cs`
- Playback: `SoundManager.StringToPhonemes()`
- Editor: `Assets/Scripts/synth/`

### Inspector Wiring
**Critical**: Heavy use of Inspector-based wiring (BetterEvents, ScriptableObject Events, UnityEvents). Code alone won't show full behavior—check prefabs and scenes for event connections.

## Story Data Formats

### Runtime Format (`.rody.json`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LaCreArthur) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->

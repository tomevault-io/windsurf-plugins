---
trigger: always_on
description: Unity editor extension for converting VRChat PC avatars to Android (Quest/PICO) platform.
---

# VRCQuestTools

Unity editor extension for converting VRChat PC avatars to Android (Quest/PICO) platform.

## Commands

### Unity Editor Operations (via uloop skills)

**Preferred** for compilation and testing when Unity Editor is running. Uses [unity-cli-loop](https://github.com/hatayama/unity-cli-loop)-based skills:

> First run `uloop-launch` to start Unity, then run other `uloop-*` commands after the editor is ready.

| Skill | Purpose |
|-------|---------|
| `uloop-launch` | Launch Unity project with matching Editor version |
| `uloop-compile` | Compile the project and report errors/warnings |
| `uloop-run-tests` | Run Unity Test Runner (EditMode/PlayMode) and get results |
| `uloop-get-logs` | Retrieve Unity Console logs |
| `uloop-clear-console` | Clear Unity Console before a fresh run |

Note: When you need to execute `npm` and `npx`, use `pnpm` and `pnpm dlx` instead.

### Lint

| Command | Description |
|---------|-------------|
| `pwsh scripts/lint.ps1` | Lint on Windows (builds .sln then checks VRCQuestTools*.csproj) |
| `bash scripts/lint.sh` | Lint on Linux CI |

### Fallback (no Unity Editor)

> `dotnet build VRCQuestTools.sln` — build only; requires Unity-generated .sln/.csproj files (Unity 2022.3.22f1). Tests are not runnable without Unity; CI uses `game-ci/unity-test-runner`.

### Website (Documentation)

| Command | Description |
|---------|-------------|
| `cd Website && pnpm install --frozen-lockfile` | Install docs dependencies |
| `cd Website && pnpm run build` | Build documentation site (Docusaurus) |

## Architecture

```
Packages/com.github.kurotu.vrc-quest-tools/   # Main VPM package
  Editor/                                      # Editor-only code (bulk of the project)
    Automators/        # Automated avatar conversion tasks
    I18n/              # Internationalization (en-US, ja-JP, ru-RU .po files)
    Inspector/         # Custom Inspector/PropertyDrawer UI classes
    Models/            # Data models and business logic
      MaterialGenerators/  # Per-shader material conversion
      Unity/               # Unity object wrappers
      VRChat/              # VRChat-specific models (avatar, dynamics)
    NDMF/              # Non-Destructive Modular Framework integration
      Passes/            # NDMF processing passes
      Errors/            # NDMF error reporting
    Services/          # Business logic services
    Utils/             # Utility classes (texture, SDK, animation, etc.)
    ViewModels/        # MVVM ViewModels
    Views/             # Editor windows and dialogs
  Runtime/                                     # Runtime components (on avatars)
    Components/        # MonoBehaviour components (AvatarConverterSettings, etc.)
    Models/            # Runtime data models (BuildTarget, TextureFormat)
  Shader/              # Custom shaders for material baking
    cginc/             # Shared shader includes
  ComputeShader/       # Compute shaders (normal map downsampling)
  Assets/              # Precomputed resources (blank normal maps)

Assets/
  VRCQuestTools-Tests/         # Test suite
    Editor/                    # NUnit editor tests
      Models/                  # Model tests
      Utils/                   # Utility tests
      Services/                # Service tests
    Fixtures/                  # Test data (materials, prefabs, textures, scenes)
  VRCQuestTools-DebugUtil/     # Debug utilities (editor menu)

Website/               # Docusaurus documentation site (pnpm, bilingual en/ja)
scripts/               # CI/automation scripts (bump, lint, release)
```

## Key Files

- `Packages/com.github.kurotu.vrc-quest-tools/package.json` — VPM package metadata, version, dependencies
- `Packages/com.github.kurotu.vrc-quest-tools/Editor/VRCQuestTools.cs` — Main entry point, version constant, export method
- `Packages/com.github.kurotu.vrc-quest-tools/Editor/Models/VRChat/VRChatAvatar.cs` — Core avatar model
- `Packages/com.github.kurotu.vrc-quest-tools/Editor/Utils/TextureUtility.cs` — Texture baking utilities
- `Packages/com.github.kurotu.vrc-quest-tools/Editor/Utils/VRCSDKUtility.cs` — VRChat SDK integration
- `Assets/Default.ruleset` — StyleCop analyzer rule suppressions

## Code Style

- **Namespaces:** `KRT.VRCQuestTools.<Feature>` (e.g., `KRT.VRCQuestTools.Models`, `KRT.VRCQuestTools.Utils`)
- **Access modifiers:** All types are `internal` unless they must be public
- **File headers:** C# source files in the main packages should include a copyright header (`// <copyright file="..." company="kurotu">`); new or modified files must follow this pattern, though some older files may not yet be updated.
- **Indentation:** 4 spaces for C# (`.editorconfig`)
- **Line endings:** Preserve each existing file's current line endings when editing. Only newly added files should use LF (`\n`). Do not bulk-convert existing CRLF files to LF because this repository still contains many CRLF files and mass normalization creates large, noisy diffs.
- **XML docs:** `///` on all public/internal members with `<summary>`, `<param>`, `<returns>`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kurotu/VRCQuestTools](https://github.com/kurotu/VRCQuestTools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->

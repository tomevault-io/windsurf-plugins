---
trigger: always_on
description: **ALL keyboard commands (Cmd+[, Cmd+L, Cmd+T, Cmd+R, etc.) DO NOT WORK with iOS apps via iPhone Mirroring.**
---

## CRITICAL: Keyboard Commands DO NOT Work in iOS Apps

**ALL keyboard commands (Cmd+[, Cmd+L, Cmd+T, Cmd+R, etc.) DO NOT WORK with iOS apps via iPhone Mirroring.**

iPhone Mirroring only passes through to iOS apps:
- **tap** — touch at coordinates
- **swipe** — scroll gestures
- **drag** — touch-and-drag (rearranging, sliders)
- **long press** — context menus
- **double tap** — zoom, text selection
- **type_text** — character input (ONLY when a text field is active on the iPhone)

**Back navigation**: The ONLY way to go back in an iOS app is to OCR-detect the "<" back chevron in the top 15% of screen and tap it. `press_key(key: "[", modifiers: ["command"])` does NOT work. The explorers use `tapBackButton()` for this.

**press_key with modifiers**: Only works for Mac-level actions (e.g., shake via Ctrl+Cmd+Z). iOS apps do not receive keyboard shortcuts through iPhone Mirroring.

---

## CRITICAL: MCP Restart After Code Changes

After modifying any Swift source files, the running MCP server still uses the **old binary**. You MUST ask the user to run `/mcp` to restart the server before testing with MCP tools (`generate_skill`, `tap`, `describe_screen`, etc.). Without a restart, your code changes have no effect on MCP tool behavior.

---

## Sibling Repositories

This project has companion repos on the same machine. Reference them when needed:

| Repo | Local Path | Purpose |
|------|-----------|---------|
| [mirroir-skills](https://github.com/jfarcand/mirroir-skills) | `../mirroir-skills` | Community skill YAML files (apps/, workflows/, testing/) |

Compiled `.compiled.json` files live alongside their source `.yaml` in the skills repo.

## Setup After Clone

```bash
git config core.hooksPath git-hooks
```

This activates the `commit-msg` hook in `git-hooks/` which enforces conventional commit format, max 2-line messages, and rejects `Co-Authored-By: Claude` lines.

## Package Manager: Swift Package Manager

This project uses **Swift Package Manager** (SPM) exclusively. The `Package.swift` manifest defines all targets and dependencies.

### Commands
| Task | Command |
|------|---------|
| Build | `swift build` |
| Build release | `swift build -c release` |
| Run tests | `swift test` |
| Clean | `swift package clean` |
| Resolve dependencies | `swift package resolve` |

## Git Workflow: NO Pull Requests

**CRITICAL: NEVER create Pull Requests. All merges happen locally via squash merge.**

### Rules
- **NEVER use `gh pr create`** or any PR creation command
- **NEVER suggest creating a PR**
- Feature branches are merged via **local squash merge**

### Workflow for Features
1. Create feature branch: `git checkout -b feature/my-feature`
2. Make commits, push to remote: `git push -u origin feature/my-feature`
3. When ready, squash merge locally (from main worktree):
   ```bash
   git checkout main
   git fetch origin
   git merge --squash origin/feature/my-feature
   git commit
   git push
   ```

### Bug Fixes
- Bug fixes go directly to `main` branch (no feature branch needed)
- Commit and push directly: `git push origin main`

## Architecture

This project follows established decomposition patterns. When adding new functionality, match these patterns — do not invent new structural idioms.

### File Size Limit

No file should exceed **500 lines**. If a type is growing past this threshold, extract a focused helper type or enum. Reference: `LandmarkPicker` and `ActionStepFormatter` were extracted from `SkillMdGenerator` for this reason.

### Pattern Catalog

Apply the pattern whose trigger condition matches your situation:

| Trigger | Pattern | Example |
|---------|---------|---------|
| New MCP tool category | **Extension-Based Tool Registration**: create `XxxTools.swift` with `registerXxxTools()`, wire from `ToolHandlers.registerTools()` | `InputTools.swift`, `ScreenTools.swift` |
| Tool handler needs business logic | **Thin Registration → Delegate**: tool file owns schema + arg parsing only; separate type owns logic | `InputTools.swift` → `InputSimulation.swift` |
| New system boundary (hardware, OS API, network) | **Protocol Abstraction**: define protocol in `Protocols.swift`, implement in concrete type | `WindowBridging`/`MirroringBridge`, `InputProviding`/`InputSimulation` |
| Pure transformation (filter, format, match, compute) | **Enum Namespace**: stateless enum, all `static func`, no stored properties | `LandmarkPicker`, `ActionStepFormatter`, `ElementMatcher` |
| Multi-step stateful workflow (start/accumulate/finalize) | **Session Accumulator**: `final class` with `NSLock`, explicit lifecycle methods | `ExplorationSession` |
| Wrapping a protocol to add observation/caching | **Decorator**: new type conforming to same protocol, forwarding + adding behavior | `RecordingDescriber` wraps `ScreenDescribing` |
| Two input formats producing same output type | **Separate Parsers, Shared Model**: one parser per format, both emit same type | `SkillParser` (YAML) + `SkillMdParser` (Markdown) → `SkillStep` |
| Generator building structured output from data | **Pipeline with Composable Stages**: generator delegates filtering/formatting to enum-namespace helpers | `SkillMdGenerator` uses `LandmarkPicker` + `ActionStepFormatter` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jfarcand/mirroir-mcp](https://github.com/jfarcand/mirroir-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->

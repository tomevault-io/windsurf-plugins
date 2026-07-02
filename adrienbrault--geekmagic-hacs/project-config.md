---
trigger: always_on
description: Home Assistant custom integration for GeekMagic displays (SmallTV Pro and similar ESP8266-based devices).
---

# GeekMagic HACS Integration

Home Assistant custom integration for GeekMagic displays (SmallTV Pro and similar ESP8266-based devices).

## Development

Use `uv` for all Python operations:

```bash
uv sync                       # Install dependencies
uv run pytest                 # Run tests
uv run pytest -v              # Run tests with verbose output
uv run ruff check .           # Lint code
uv run ruff format .          # Format code
uv run ty check               # Type check
uv run pre-commit run --all   # Run all pre-commit hooks
```

## Git Workflow

Follow **Conventional Commits** and create **atomic commits** as you work:

### Commit Types
- `feat:` New feature
- `fix:` Bug fix
- `refactor:` Code refactoring (no functional change)
- `docs:` Documentation only
- `test:` Adding/updating tests
- `chore:` Maintenance (deps, config, tooling)
- `style:` Formatting, whitespace (no code change)

### Atomic Commits
Create small, focused commits that each represent a single logical change:

1. **After implementing a feature** → commit the feature
2. **After fixing a bug** → commit the fix
3. **After adding tests** → commit the tests
4. **After refactoring** → commit the refactor

**Always run pre-commit before committing**: `uv run pre-commit run --all`

This validates tests, linting, formatting, and type checking in one command.

### Examples
```bash
git commit -m "feat: add clock widget with timezone support"
git commit -m "fix: handle missing entity gracefully in EntityWidget"
git commit -m "test: add unit tests for sparkline rendering"
git commit -m "refactor: extract color parsing into helper function"
git commit -m "chore: add ty type checker and pre-commit hooks"
```

## Release Process

HACS detects new versions via GitHub releases. The user creates releases in the GitHub UI (with auto-generated notes); Codex prepares the version bump.

### When the user asks for a version bump (e.g. "bump to 1.0.1", "release a new patch")

1. Determine the new version using semver:
   - **Patch** (`1.0.0 → 1.0.1`): bug fixes only
   - **Minor** (`1.0.0 → 1.1.0`): new features, backward-compatible
   - **Major** (`1.0.0 → 2.0.0`): breaking changes
2. Update `version` in `custom_components/geekmagic/manifest.json`
3. Commit on `main` (or a `chore/bump-X.Y.Z` branch if a PR is preferred):
   ```
   chore: bump version to X.Y.Z
   ```
4. Push, then tell the user to create the release in GitHub UI:
   - Releases → "Draft a new release"
   - Tag: `vX.Y.Z` (matches `manifest.json`)
   - Target: the bump commit on `main`
   - Click "Generate release notes" → Publish

### Critical rules
- **Tag must match `manifest.json` version exactly** (HA core reads `manifest.json` and a mismatch breaks update detection)
- **Tag the bump commit, not an earlier one** — otherwise the tagged tree still has the old version
- Tag format: `vX.Y.Z` (with leading `v`)
- Never tag or create the release yourself — the user does that in GitHub UI

## Project Structure

```
custom_components/geekmagic/
├── __init__.py       # Integration entry, services
├── config_flow.py    # Device setup + options flow
├── coordinator.py    # Data update coordinator
├── device.py         # HTTP API client for GeekMagic
├── renderer.py       # Pillow image generation
├── const.py          # Constants and config keys
├── widgets/          # Widget components
│   ├── base.py       # Widget base class
│   ├── clock.py      # Clock widget
│   ├── entity.py     # HA entity display
│   ├── media.py      # Media player widget
│   ├── chart.py      # Sparkline chart
│   ├── helpers.py    # Widget helper functions
│   └── text.py       # Static/dynamic text
├── layouts/          # Layout systems
│   ├── base.py       # Layout base class
│   ├── grid.py       # 2x2, 2x3, 3x3 grids
│   ├── hero.py       # Hero + footer layout
│   └── split.py      # Split panel layouts
├── entities/         # Entity platform implementations
│   ├── entity.py     # Base GeekMagicEntity class
│   ├── number.py     # Number entities (brightness, etc.)
│   ├── select.py     # Select entities (layout, widget type)
│   ├── switch.py     # Switch entities (boolean options)
│   ├── text.py       # Text entities (names, labels)
│   ├── button.py     # Button entities (refresh, nav)
│   └── sensor.py     # Sensor entities (status, dividers)
├── number.py         # Re-export for HA platform discovery
├── select.py         # Re-export for HA platform discovery
├── switch.py         # Re-export for HA platform discovery
├── text.py           # Re-export for HA platform discovery
├── button.py         # Re-export for HA platform discovery
├── sensor.py         # Re-export for HA platform discovery
├── manifest.json     # HACS metadata
└── strings.json      # UI translations
```

## Key Concepts

### Rendering Pipeline
1. Coordinator triggers update on interval
2. Layout calculates widget rectangles (slots)
3. Each widget renders into its slot using Pillow
4. Image converted to JPEG and uploaded to device

### Widget Interface
```python
class Widget(ABC):
    def render(self, ctx: RenderContext, hass) -> None:
        """Draw widget using the render context (local coordinates)."""

    def get_entities(self) -> list[str]:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [adrienbrault/geekmagic-hacs](https://github.com/adrienbrault/geekmagic-hacs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->

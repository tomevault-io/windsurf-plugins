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
        """Return entity IDs this widget depends on."""
```

### Layout Interface
```python
class Layout(ABC):
    def _calculate_slots(self) -> None:
        """Calculate slot rectangles."""

    def render(self, renderer, draw, hass) -> None:
        """Render all widgets in their slots."""
```

## Device API

GeekMagic devices use a simple HTTP API:

```
POST /doUpload?dir=/image/   # Upload image (multipart form)
GET  /set?img=/image/{file}  # Display image
GET  /set?theme=3            # Set custom image mode
GET  /set?brt={0-100}        # Set brightness
GET  /app.json               # Get device state
```

## Display Constraints

- Resolution: 240x240 pixels
- Physical size: ~4cm diagonal
- Minimum font size: 10-12px for readability
- Use high contrast colors (light on dark)
- JPEG upload is faster than PNG (~2.5s vs ~5.8s)

## Font Sizing System

Fonts are automatically scaled based on container height. Two naming systems are supported:

### Semantic Sizes (Preferred)

Use these for new widgets - they scale proportionally to container height:

| Size | Ratio | Use Case |
|------|-------|----------|
| `primary` | 35% | Main value (clock time, large number) |
| `secondary` | 20% | Supporting info (date, unit) |
| `tertiary` | 12% | Labels, captions |

```python

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [adrienbrault/geekmagic-hacs](https://github.com/adrienbrault/geekmagic-hacs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->

---
trigger: always_on
description: This file provides detailed guidance for working with the bignumber-card-continued project.
---

# CLAUDE.md - Big Number Card Continued

This file provides detailed guidance for working with the bignumber-card-continued project.

## Project Overview

Community-maintained continuation of the original bignumber-card by [@ciotlosm](https://github.com/ciotlosm). The original project has not been updated since January 2022. This continuation aims to maintain compatibility, fix bugs, and incorporate valuable community contributions.

## Continuation Philosophy

Following the same principles as canvas-gauge-card-continued in this monorepo:

1. **Respect Original Work**: Credit original authors, maintain git history when possible
2. **Prioritize Stability**: Bug fixes and compatibility updates before new features
3. **Community-Driven**: Review and merge valuable PRs from the original repository
4. **Conservative Versioning**: Use `-continued` suffix (e.g., `0.0.7-continued` or `1.0.0-continued`)
5. **Maintain Simplicity**: Keep the single-file vanilla JS approach - no build system needed

## Architecture

### Technology Stack

- **Framework**: Vanilla JavaScript Web Components (no dependencies)
- **Build System**: None required
- **File Structure**: Single file (`bignumber-card.js`)
- **Browser Support**: Modern browsers with Web Components support

### Code Structure

**Class**: `BigNumberCard extends HTMLElement`

**Key Methods**:
- `constructor()`: Creates Shadow DOM attachment
- `setConfig(config)`: Validates config, builds DOM structure, applies initial styles
- `set hass(hass)`: Called on every Home Assistant state update, updates displayed values
- `getCardSize()`: Returns card height hint (always returns 1)

**Helper Methods**:
- `_fire(type, detail, options)`: Dispatches custom events (used for more-info dialog)
- `_computeSeverity(stateValue, sections)`: Finds matching severity level
- `_getColor(entityState, config)`: Resolves text color (severity → config → default)
- `_getStyle(entityState, config)`: Resolves background color (severity → config → default)
- `_translatePercent(value, min, max)`: Converts value to progress bar percentage

### Rendering Approach

**Shadow DOM with Direct Manipulation**:
1. `setConfig()` builds DOM structure once:
   - Creates `<ha-card>` container
   - Adds `#value` div for the number display
   - Adds `#title` div for the title
   - Injects `<style>` tag with CSS custom properties
2. `set hass()` updates content via direct DOM manipulation:
   - Modifies CSS custom properties for dynamic styling
   - Updates text content of value/title elements
   - Applies/removes CSS classes for None state handling

**Why This Approach**:
- Lightweight: ~150 lines, no dependencies
- Fast: No virtual DOM overhead for such simple updates
- Compatible: Uses standard Web Components APIs

### CSS Architecture

**CSS Custom Properties** (set dynamically):
- `--bignumber-color`: Text color
- `--bignumber-fill-color`: Background/bar color
- `--bignumber-percent`: Progress bar fill percentage (100% = empty, 0% = full)
- `--bignumber-direction`: Bar fill direction (left/right/top/bottom)
- `--base-unit`: Base scale for all sizing (default: 50px)

**Gradient Background**:
Uses CSS linear-gradient to create progress bar effect:
```css
background: linear-gradient(
  to var(--bignumber-direction),
  var(--card-background-color) var(--bignumber-percent),
  var(--bignumber-fill-color) var(--bignumber-percent)
);
```

## Configuration System

### Required Options
- `entity`: Sensor entity ID (e.g., `sensor.temperature`)

### Display Options
- `title`: Card title text
- `scale`: Base unit for sizing (default: "50px")
- `attribute`: Entity attribute to display instead of state
- `hideunit`: Boolean to hide unit of measurement
- `round`: Number of decimal places for rounding

### Styling Options
- `color`: Text color (hex or CSS variable)
- `bnStyle`: Background/bar color (hex or CSS variable)
- `opacity`: Opacity for unit text (default: "0.5")

### Progress Bar Options
- `min`: Minimum value (enables bar display)
- `max`: Maximum value (required if min is set)
- `from`: Fill direction - "left", "right", "top", or "bottom" (default: "left")

### Severity System
Array of objects defining value-based color changes:
```javascript
severity: [
  { value: 70, bnStyle: 'var(--label-badge-green)' },
  { value: 90, bnStyle: 'var(--label-badge-yellow)', color: '#000' },
  { value: 100, bnStyle: 'var(--label-badge-red)', color: '#FFF' }
]
```

**Rules**:
- Must be in ascending order by `value`
- Values represent upper thresholds
- First matching threshold is used
- Can override both `bnStyle` and `color`

### None/Offline State Handling
Special configuration for handling None/unavailable sensors:
- `noneString`: Display text when value is None (e.g., "Offline")
- `noneCardClass`: CSS class to add to `<ha-card>` when None
- `noneValueClass`: CSS class to add to `#value` when None

## Known Issues

### Bug in Original Code
**Line 21**: Typo in property name
```javascript
if (!cardConfig.noneString) cardConfig.nonestring = null;
```
Should be `noneString` (capital S) for consistency. This bug doesn't break functionality but indicates code quality issues.

### Missing Error Handling
**Line 115**: No validation before accessing entity
```javascript

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sxdjt/bignumber-card-continued](https://github.com/sxdjt/bignumber-card-continued) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->

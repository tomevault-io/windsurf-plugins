---
trigger: always_on
description: > Work in Progress! The following might change at any moment.
---

> Work in Progress! The following might change at any moment.

## Project Structure Overview

**Single-file architecture.** The whole card lives in `/home/user/hass-spatial-lights-card/hass-spatial-lights-card.js`. It's a Web Component that extends `HTMLElement` (`class SpatialLightColorCard`), plus a sibling editor (`class SpatialLightColorCardEditor`).

> ℹ️ Anchors below are **function/identifier names** rather than line numbers — line numbers drift constantly. Use `grep -n` to jump to a name (e.g. `grep -n '_renderAll' hass-spatial-lights-card.js`).

**Key concepts:**
- `setConfig(config)` — Lovelace contract; normalizes config and re-renders if `hass` is set.
- `set hass(hass)` — Lovelace contract; diffs `prev.states` vs `next.states` against the config's entity set (`_isRelevantHassChange`) and runs `updateLights()` only when something relevant changed. First call goes through `_renderAll()`.
- `_renderAll()` — destructive: writes the whole `shadowRoot.innerHTML`, caches element refs, calls `_attachEventListeners()`. Calls `_cancelActiveInteractions()` first so an in-flight gesture commits its pending value before the DOM is wiped.
- `updateLights()` — non-destructive: walks each `.light` and toggles classes / sets styles based on current state (color, on/off, selected, unavailable). Drives `_updateControlValues`, `_refreshColorPresets`, `_updateAllGlows`, `_repositionLabels`.

---

## 1. Slider Controls Rendering

**Renderers:** `_renderControlsFloating(visible, controlContext)` and `_renderControlsBelow(controlContext)`. Both emit the same children: a 256×256 mini color wheel canvas, two `<input type="range">` sliders (brightness 0-255, temperature in `tempRange.min..max`), and the presets area.

**Layout modes:**
- Desktop (`@media (min-width: 769px)`): CSS Grid 2×2. Color wheel `grid-column: 1; grid-row: 1/3`; sliders top-right; presets bottom-right wrap.
- Mobile (`@media (max-width: 768px)`): flex-wrap row. Wheel order 1, presets order 2 with `max-width: calc(100% - 140px)`, sliders order 3 at full width.
- Floating vs below: `controls_below: true` (default) renders the controls block after the canvas; otherwise they're absolutely positioned over the canvas with mobile-aware insets.

**Slider gesture (`_bindSliderGesture`):** Pointer-down captures and immediately calls `_applyPointerValue(el, clientX)`. Pointermove follows the finger, with a 6 px / `dy > dx` heuristic that releases capture and reverts the value when the user is actually trying to scroll the page. Pointerup commits via `_handleBrightnessChange()` or `_handleTemperatureChange()`. The active gesture is recorded in `this._activeSliderGesture` so `_updateControlValues` skips clobbering the value while the user's finger is down.

**Visual updates:**
- `_updateSliderVisual(el)` — sets `--slider-percent` / `--slider-ratio` from `value`/`min`/`max`.
- `_updateControlValues(controlContext)` — full sync to averaged state, plus capability gating via `_getControlCapabilities()` (toggles `disabled` attribute on sliders, `.disabled` on color wheel, `.no-rgb-support`/`.no-temp-support`/`.no-brightness-support` on the controls container). The temperature slider's warm-to-cool gradient is a static CSS background — it's a visual affordance, not a precise color readout.

---

## 2. Color Presets & Live Colors

**Flow:** `_renderPresetsContent()` → `_renderColorPresets()` + (separator) + `_renderTemperaturePresets()` + `_renderEffectPresets()`.

**Color presets (`_renderColorPresets`):**
1. `_getLiveColors()` walks `state === 'on'` entities, filters by `color_mode` ∈ `RGB_COLOR_MODES`, deduplicates with `_rgbDistance() < COLOR_TOLERANCE` (constants on the class).
2. Filters live colors against config presets so config wins on collision.
3. Each preset is a focusable `<div role="button" tabindex="0">` carrying `data-preset-color`, optionally `data-preset-rgb`, and `data-preset-entities`.

**Active state:** `_getActivePresetColor()` returns the unique RGB shared by all controlled lights (within tolerance), or null. The preset whose RGB matches gets `.active`.

**Temperature presets (`_renderTemperaturePresets`):** Only emitted if `show_live_colors: true`. `_getLiveTemperatures()` groups lights with `color_mode === 'color_temp'` by ±`TEMP_TOLERANCE` Kelvin. The swatch is `_kelvinToRgb(kelvin)` (Tanner Helland, clamped to 1000-40000 K).

**Separator:** A 1×20 px div between RGB and temp presets. `_updateSeparatorVisibility()` checks via `getBoundingClientRect()` whether the previous color preset and the next temp preset are on the same row, and hides the separator otherwise.

**Preset interaction:** `_bindPresetHighlight(el)` and `_bindPresetHandlers()`. Mouse hover highlights matching lights via `pointerenter`/`pointerleave`; touch uses a 300 ms long-press to highlight, then clears on `pointerup`. Click applies via `_applyColorWheelSelection(rgb)` / `_applyTemperaturePreset(kelvin)` / `_applyEffectPreset(effect)`. Keyboard Enter/Space activates from `_handleKeyDown` → `composedPath()` lookup.

---

## 3. Light Rendering

**`_renderLightsHTML()`** maps over `_config.entities`, renders each as:

```html
<div class="light {state} {selected} {iconOnly} {unavailable}"
     style="left:Px%; top:Px%; ..."

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Mihonarium/hass-spatial-lights-card](https://github.com/Mihonarium/hass-spatial-lights-card) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->

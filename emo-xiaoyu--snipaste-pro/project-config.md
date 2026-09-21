---
trigger: always_on
description: Run the local server yourself and open the preview in the browser available to this environment. Do not give the user server-start instructions when you can run it.
---

# Prototype Instructions

Run the local server yourself and open the preview in the browser available to this environment. Do not give the user server-start instructions when you can run it.

Before making substantial visual changes, use the Product Design plugin's `get-context` skill when the visual source is unclear or no longer matches the current goal. When the user gives durable prototype-specific design feedback, preferences, or decisions, record them in `AGENTS.md`.

When implementing from a selected generated mock, treat that image as the source of truth for layout, component anatomy, density, spacing, color, typography, visible content, and hierarchy.

The selected visual direction is the first generated option: a warm off-white, single-column Spotlight-style clipboard overlay with muted sage accents, lightweight list separators, text and image history rows, and keyboard-first controls.

Pasting must feel immediate: keep the local click-to-paste path free of per-action process startup and target roughly 35ms before sending Ctrl+V. Remote-control windows may use a separate longer synchronization delay so their compatibility does not slow normal local pasting.

Screenshot workflow: triggering capture freezes the current mouse display and creates a compact cursor-centered preselection on the full-screen canvas. That preselection remains freely movable, resizable from edges/corners, or replaceable until the user chooses an annotation tool or executes save/copy/pin. Ctrl+C copies, Ctrl+P pins, Enter saves. Capture, quick capture-and-pin, and screenshot-history global shortcuts are user-configurable and must be visible at the top of settings with a labeled main-window entry.

Initial selection tracking: while the cursor-centered preselection is still only a suggestion, it follows the live mouse position and clamps to the captured display edges. The first normal left-button press stops tracking; dragging from that press replaces the suggestion with a custom rectangle, while a click without a meaningful drag keeps the current rectangle fixed. Returning to the automatic suggestion with right-click resumes tracking.

Snipaste-style reference interaction: capture must start with a cursor-centered 520 × 320 preselection and must not auto-select the foreground window. Keep the selection freely replaceable before annotation, support 1px arrow-key movement and Shift+arrow resizing, show live pixel color and selection dimensions, and provide pen, arrow, rectangle, ellipse, color, width, undo, and redo controls. Enter or double-click copies and finishes; F4 copies and pins; right-click steps back. Follow the interaction model without copying Snipaste branding or assets.

Selection correction: the automatic selection is only a starting suggestion. A normal left-button drag anywhere on the frozen screen, including inside the suggested selection, must start a new rectangle from that pointer position. Edge and corner drags resize; Alt+drag inside moves the whole selection. F4 must both copy the final crop to the clipboard and pin it to the desktop.

Exact pin fidelity: a newly captured F4 pin must place its image content at the original selection's desktop x/y and logical width/height, converting from captured physical pixels using the active display bounds. A transparent outer margin may hold the distinguishing shadow only when the pin window is offset and expanded by the same margin, so the image pixels still visually complete the frozen source page at 1:1 desktop scale. Never use contain-fit letterboxing. After creation, the mouse wheel zooms around the pointer while preserving aspect ratio, and dragging moves the pin.

Screenshot launch performance: keep the screenshot editor window preloaded and hidden between captures. Do not query foreground-window bounds for initial selection. Skip the hide delay when Pasty UI is already hidden, use only one compositor frame when visible Pasty UI must be removed, and never show the editor until the frozen image has decoded and rendered. Saving or F4 pinning must render the crop directly from the frozen image and annotations without waiting for selection-overlay animation frames. Pins may use a subtle shadow margin only if the image content retains its original position and size.

Reliability correction: the preloaded screenshot renderer must explicitly announce readiness only after its `screenshot:begin` listener is registered; main must queue the latest capture payload until that handshake arrives. For exact pins, use Electron content bounds at the mapped selection rectangle rather than compensating with a CSS shadow margin or assuming frameless resizable window bounds equal web-content bounds.

Hidden-renderer correction: the preloaded screenshot window must set `backgroundThrottling: false`, because its image-ready handshake uses render frames while the window is hidden. Keep a short idempotent timer fallback so capture cannot remain invisible if the compositor does not schedule a hidden frame.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [emo-xiaoyu/snipaste-pro](https://github.com/emo-xiaoyu/snipaste-pro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-21 -->

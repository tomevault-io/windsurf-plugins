---
trigger: always_on
description: Web Interface Guidelines
---


Web Interface Guidelines

Interfaces succeed because of hundreds of choices. This is a living, non-exhaustive list of those decisions. Most guidelines are framework-agnostic; some are specific to React/Next.js. Feedback is welcome.

## Interactions

- Keyboard
  - MUST: All flows are keyboard-operable, following [WAI-ARIA APG](https://www.w3.org/WAI/ARIA/apg/patterns/).
  - MUST: Every focusable element shows a visible focus ring. Prefer `:focus-visible` over `:focus` to avoid distracting pointer users; use `:focus-within` for groups.
  - MUST: Manage focus (trap, move, and return) per APG patterns or WAI-ARIA guidelines.

- Targets & Input
  - MUST: Match visual and hit target. Visual targets <24px MUST have hit area expanded to ≥24px. On mobile, hit target MUST be ≥44px.
  - MUST: Mobile `<input>` uses font-size ≥16px or set:
    ```html
    <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, viewport-fit=cover">
    ```
  - NEVER: Disable browser zoom.
  - MUST: Set `touch-action: manipulation` to prevent double-tap zoom.
  - MUST: Set `-webkit-tap-highlight-color` to follow design.
  - MUST: Design forgiving interactions—generous hit targets, clear affordances, avoid finickiness.

- Inputs & Forms (Behavior)
  - MUST: Hydration-safe inputs (no lost focus or value after hydration).
  - NEVER: Block paste in `<input>` or `<textarea>`.
  - MUST: Loading buttons show a spinner and keep original label.
  - MUST: Enter submits focused text input if possible. In `<textarea>`, ⌘/Ctrl+Enter submits, Enter inserts a newline.
  - MUST: Keep submit enabled until request starts; then disable, show spinner, and use idempotency key.
  - MUST: Do not block typing—accept free text, validate after input.
  - MUST: Allow incomplete forms to submit and surface validation.
  - MUST: Errors inline next to fields; on submit, focus first error.
  - MUST: Use `autocomplete` with meaningful `name`, and correct `type` and `inputmode`.
  - MUST: Every control has a `<label>` or is associated with a label for a11y.
  - MUST: Clicking a `<label>` focuses corresponding input.
  - SHOULD: Disable spellcheck for emails, codes, usernames.
  - SHOULD: Placeholders end with ellipsis and show example pattern (eg, `+1 (123) 456-7890`, `sk-012345…`).
  - MUST: Warn on unsaved changes before navigation.
  - MUST: Compatible with password managers and 2FA; do not block one-time code paste.
  - MUST: Trim values to handle input method expansion/trailing spaces.
  - MUST: No input dead zones—checkbox or radio label/control share one generous hit target.
  - SHOULD: Allow submitting incomplete forms for validation feedback.
  - MUST: Autocomplete and autofill fields appropriately.
  - MUST: On Windows, set background-color and color for `<select>` to avoid dark-mode system style bugs.

- State & Navigation
  - MUST: URL reflects state (deep-link filters, tabs, pagination, expanded panels). Prefer libs like [nuqs](https://nuqs.dev).
  - MUST: Back/Forward restores previous scroll.
  - MUST: Links are semantic—use `<a>`/`<Link>` for navigation (supports Cmd/Ctrl/Right/Middle-click).
  - MUST: Scroll positions persist.

- Feedback
  - SHOULD: Optimistic UI—update immediately, reconcile on server, show rollback or undo on errors.
  - MUST: Confirm destructive actions, or provide an Undo window.
  - MUST: Use polite `aria-live` on toasts and inline validation.
  - SHOULD: Use ellipsis (`…`) for options that open follow-ups (e.g., "Rename…") or loading ("Loading…", "Saving…").
  - MUST: Announce async updates with ARIA live regions.

- Touch/Drag/Scroll
  - MUST: Delay first tooltip in a group; no delay on subsequent tooltips.
  - MUST: Use intentional `overscroll-behavior: contain` for modals/drawers.
  - MUST: During drag, disable text selection and set `inert` on dragged element/containers.
  - MUST: No “dead-looking” zones—if it looks clickable, it is.
  - MUST: Clean drag interactions—prevent unwanted selection and hover during drag.

- Autofocus
  - SHOULD: Use autofocus on desktop when there’s a single primary input. Rarely use on mobile to avoid keyboard shift.

- Shortcuts
  - SHOULD: Localize keyboard shortcuts.
  - SHOULD: Show platform-specific symbols.

## Animation

- MUST: Honor `prefers-reduced-motion`. Provide a reduced-motion variant.
- SHOULD: Prefer CSS > Web Animations API > JavaScript libraries (eg, motion).
- MUST: Animate compositor-friendly properties (`transform`, `opacity`). Avoid animating `top`, `left`, `width`, `height`.
- SHOULD: Animate only to clarify cause/effect or for deliberate delight.
- SHOULD: Choose easing to match the change (size/distance/trigger).
- MUST: Animations are interruptible and input-driven—avoid autoplay.
- MUST: Set correct `transform-origin` (motion starts where it “physically” should).
- NEVER: `transition: all`—always specify only necessary properties.
- SHOULD: For SVG, animate `<g>` wrappers and set `transform-box: fill-box; transform-origin: center` for cross-browser correctness.

## Layout

- MUST: Use `size-` utilities (eg, `size-4`) instead of `w-`/`h-` for size in Tailwind CSS.
- NEVER: Use `space-x-`, `space-y-`, or any hardcoded Tailwind utilities like `mb`, `mr`, `ml`, `mt`, etc. for spacing.  

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [preetsuthar17/HextaUI](https://github.com/preetsuthar17/HextaUI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->

---
trigger: always_on
description: Carousel component accessibility compliance pattern
---

# Carousel Accessibility Standards

Ensures carousel components follow WCAG compliance and WAI-ARIA Carousel Pattern specifications.

<rule>
name: carousel_accessibility_standards
description: Enforce carousel component accessibility standards and WAI-ARIA Carousel Pattern compliance
filters:
  - type: file_extension
    pattern: "\\.(vue|jsx|tsx|html|liquid|php|js|ts)$"

actions:
  - type: enforce
    conditions:
      # Carousel container role requirement
      - pattern: "(?i)<(div|section)[^>]*(?:carousel|slider|slideshow)[^>]*>"
        pattern_negate: "(role=\"(region|group)\"|aria-roledescription=\"carousel\")"
        message: "Carousel container must have role='region' or role='group' and aria-roledescription='carousel'."

      # Carousel label requirement
      - pattern: "(?i)<[^>]*role=\"(region|group)\"[^>]*aria-roledescription=\"carousel\"[^>]*>"
        pattern_negate: "(aria-labelledby|aria-label)=\"[^\"]+\""
        message: "Carousel must have either aria-labelledby or aria-label for accessibility."

      # Slide role requirement
      - pattern: "(?i)<(div|section)[^>]*(?:slide|carousel-item)[^>]*>"
        pattern_negate: "(role=\"group\"|aria-roledescription=\"slide\")"
        message: "Slide containers must have role='group' and aria-roledescription='slide'."

      # Slide label requirement
      - pattern: "(?i)<[^>]*role=\"group\"[^>]*aria-roledescription=\"slide\"[^>]*>"
        pattern_negate: "(aria-labelledby|aria-label)=\"[^\"]+\""
        message: "Slides must have either aria-labelledby or aria-label for accessibility."

      # Rotation control requirement
      - pattern: "(?i)<button[^>]*(?:rotation|auto-play|autoplay)[^>]*>"
        pattern_negate: "aria-label=\"[^\"]*(?:Start|Stop)[^\"]*slide[^\"]*rotation[^\"]*\""
        message: "Rotation control must have aria-label indicating its current state (Start/Stop slide rotation)."

      # Navigation controls requirement
      - pattern: "(?i)<button[^>]*(?:next|previous|prev)[^>]*>"
        pattern_negate: "aria-label=\"[^\"]*(?:Next|Previous)[^\"]*slide[^\"]*\""
        message: "Navigation controls must have aria-label indicating their purpose (Next/Previous slide)."

      # Navigation button disabling check
      - pattern: "(?i)\\.disabled.*next|previous.*disabled"
        message: "Navigation buttons should not be disabled. Implement wrap-around navigation to first/last slide instead for better user experience."

      # Missing keyboard event handlers
      - pattern: "(?i)<button[^>]*(?:rotation|next|previous|prev)[^>]*>"
        pattern_negate: "(onKeyDown|onkeydown|@keydown|v-on:keydown)"
        message: "Carousel controls should handle keyboard events (Enter, Space)."

      # Auto-rotation interval check (WCAG 2.2.2)
      - pattern: "setInterval\\([^,]+,\\s*(?:[0-4]\\d{3}|[0-9]{1,4})\\)"
        message: "Auto-rotation interval must be at least 5000ms (5 seconds) to comply with WCAG 2.2.2 Pause, Stop, Hide."

      # Mouse hover event handlers check
      - pattern: "(?i)<(div|section)[^>]*(?:carousel|slider|slideshow)[^>]*>"
        pattern_negate: "(onMouseEnter|onmouseenter|@mouseenter|v-on:mouseenter|onMouseLeave|onmouseleave|@mouseleave|v-on:mouseleave)"
        message: "Carousel must handle mouseenter/mouseleave events to pause/resume auto-rotation."

      # aria-live attribute check
      - pattern: "(?i)<[^>]*aria-live=\"[^\"]*\"[^>]*>"
        pattern_negate: "aria-live=\"(off|polite)\""
        message: "Carousel container must have aria-live set to 'off' during rotation and 'polite' when paused."

  - type: suggest
    message: |
      **Carousel Component Accessibility Best Practices:**

      **Required ARIA Attributes:**
      - **role='region' or role='group':** Set on carousel container
      - **aria-roledescription='carousel':** Set on carousel container
      - **aria-labelledby/aria-label:** Set on carousel container
      - **role='group':** Set on slide containers
      - **aria-roledescription='slide':** Set on slide containers
      - **aria-labelledby/aria-label:** Set on slide containers
      - **aria-label:** Set on rotation control (changes with state)
      - **aria-label:** Set on navigation controls
      - **aria-live:** Set to 'off' during rotation, 'polite' when paused

      **Optional ARIA Attributes:**
      - **aria-atomic='false':** On slide wrapper
      - **aria-hidden='true':** Set on inactive slides to hide from screen readers
      - **visibility: hidden:** CSS property on inactive slides to hide from keyboard and visual users

      **Keyboard Interaction Requirements:**
      - **Tab/Shift+Tab:** Navigate through interactive elements
      - **Enter/Space:** Activate controls
      - **Auto-rotation:** Stops on focus or mouse hover, resumes on blur or mouse away
      - **Rotation Control:** First in tab sequence

      **Navigation Button Best Practices:**
      - **Always Enabled:** Navigation buttons should never be disabled
      - **Wrap-Around Navigation:** Next button wraps to first slide, Previous button wraps to last slide
      - **Consistent Behavior:** Users can always navigate in both directions
      - **Better UX:** Prevents users from getting "stuck" at slide boundaries

      **Auto-rotation Requirements (WCAG 2.2.2):**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dojoVader) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->

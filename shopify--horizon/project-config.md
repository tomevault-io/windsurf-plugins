---
trigger: always_on
description: Enforce animation accessibility standards per WCAG 2.2.2 Pause Stop Hide, 2.3.1 Three Flashes or Below Threshold, and 2.3.3 Animation from Interactions requirements
---


# Animation Accessibility Standards

Ensures animations follow WCAG compliance and provide inclusive motion design for users with different accessibility needs including photosensitivity, motion sickness, and cognitive impairments.

<rule>
name: animation_accessibility_standards
description: Enforce animation accessibility standards per WCAG 2.2.2 Pause Stop Hide, 2.3.1 Three Flashes or Below Threshold, and 2.3.3 Animation from Interactions requirements
filters:
  - type: file_extension
    pattern: "\\.(vue|jsx|tsx|html|liquid|php|js|ts|css|scss|sass|less)$"

actions:
  - type: enforce
    conditions:
      # Missing prefers-reduced-motion media query for animations
      - pattern: "(animation|transition|transform|@keyframes)"
        pattern_negate: "@media\\s*\\(prefers-reduced-motion:\\s*reduce\\)"
        message: "Animations should include prefers-reduced-motion: reduce media query to provide safer alternatives for motion-sensitive users."

      # Flashing animations exceeding 3Hz frequency
      - pattern: "animation.*(?:pulse|flash|blink|flicker)"
        pattern_negate: "animation-duration:\\s*[0-9]*\\.?[0-9]+s|animation-duration:\\s*[0-9]*\\.?[0-9]+ms"
        message: "Flashing animations must have duration ensuring frequency is below 3Hz (0.33s) to prevent seizures per WCAG 2.3.1."

      # Rapid color transitions that may trigger photosensitivity
      - pattern: "transition.*color.*[0-9]*\\.?[0-9]+s|transition.*background.*[0-9]*\\.?[0-9]+s"
        pattern_negate: "transition.*color.*[0-9]*\\.?[0-9]+s.*[0-9]*\\.?[0-9]+s|transition.*background.*[0-9]*\\.?[0-9]+s.*[0-9]*\\.?[0-9]+s"
        message: "Color transitions should be slow and smooth to avoid triggering photosensitivity. Use longer durations and easing functions."

      # Large spatial movements without reduced motion alternatives
      - pattern: "transform.*translate\\([^)]*[0-9]{2,}[^)]*\\)|transform.*translate\\([^)]*-[0-9]{2,}[^)]*\\)"
        pattern_negate: "@media\\s*\\(prefers-reduced-motion:\\s*reduce\\)"
        message: "Large spatial movements should have reduced motion alternatives using prefers-reduced-motion media query."

      # Parallax effects without user control
      - pattern: "(parallax|scroll.*jack|scroll.*hijack)"
        pattern_negate: "(prefers-reduced-motion|user.*control|pause.*animation)"
        message: "Parallax and scroll jacking effects should respect user motion preferences and provide controls to pause/disable."

      # Auto-playing animations without pause controls
      - pattern: "animation.*infinite|animation.*loop"
        pattern_negate: "(pause.*control|user.*control|prefers-reduced-motion)"
        message: "Looping animations must provide pause controls and respect prefers-reduced-motion preferences."

      # Unexpected system-triggered animations
      - pattern: "animation.*(?:appear|fade.*in|slide.*in)"
        pattern_negate: "(user.*interaction|click|hover|focus|prefers-reduced-motion)"
        message: "System-triggered animations should be subtle and respect user motion preferences."

      # Missing animation alternatives for essential UI changes
      - pattern: "(?:loading|spinner|progress|status)"
        pattern_negate: "(animation|transition|@keyframes)"
        message: "Essential UI elements like loading indicators should have appropriate animations to communicate state changes."

      # Excessive animation duration that may cause motion sickness
      - pattern: "animation-duration:\\s*[5-9]\\.[0-9]+s|animation-duration:\\s*[0-9]{2,}s"
        message: "Long animation durations may cause motion sickness. Consider shorter durations and provide reduced motion alternatives."

      # Missing focus indicators for animated interactive elements
      - pattern: "(?:button|a|input|select|textarea).*\\{[^}]*animation"
        pattern_negate: "(focus|focus-visible|outline|box-shadow)"
        message: "Animated interactive elements must have visible focus indicators for keyboard navigation accessibility."

      # Animation without meaningful purpose or context
      - pattern: "animation.*(?:bounce|wiggle|shake|rotate)"
        pattern_negate: "(loading|status|feedback|interaction)"
        message: "Animations should serve a meaningful purpose. Avoid decorative animations that may distract or confuse users."

      # Missing animation state management
      - pattern: "animation.*(?:play|pause|stop)"
        pattern_negate: "(prefers-reduced-motion|user.*control|aria.*live)"
        message: "Animation state changes should be communicated to assistive technology and respect user preferences."

  - type: suggest
    message: |
      **Animation Accessibility Best Practices:**

      **1. Respect Motion Preferences (WCAG 2.3.3):**
      ```css
      /* Default animation */
      .fade-in {
        animation: fadeIn 0.3s ease-in-out;
      }

      /* Reduced motion alternative */
      @media (prefers-reduced-motion: reduce) {
        .fade-in {
          animation: none;
          opacity: 1;
        }
      }
      ```

      **2. Seizure Prevention (WCAG 2.3.1):**
      ```css
      /* Safe flashing animation - below 3Hz threshold */
      .pulse {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Shopify/horizon](https://github.com/Shopify/horizon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->

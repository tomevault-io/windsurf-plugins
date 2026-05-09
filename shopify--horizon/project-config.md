---
trigger: always_on
description: Mobile accessibility standards per WCAG 2.5.8 Target Size (Minimum), 2.4.1 Bypass Blocks, and 1.3.4 Orientation requirements
---


# Mobile Accessibility Best Practices

Ensures mobile interfaces follow WCAG compliance and provide proper accessibility for touch interactions, spacing, and orientation flexibility.

<rule>
name: mobile_accessibility_standards
description: Enforce mobile accessibility standards per WCAG requirements for touch targets, spacing, and orientation
filters:
  - type: file_extension
    pattern: "\\.(vue|jsx|tsx|html|liquid|php|js|ts|css|scss|sass|less)$"

actions:
  - type: enforce
    conditions:
      # Touch targets smaller than minimum recommended size
      - pattern: "(width|height):\\s*(?:1[0-9]|2[0-3])\\s*(?:px|rem|em|%)"
        pattern_negate: "(min-width|min-height|max-width|max-height)"
        message: "Touch targets should be at least 24x24px (Level AA) or 44x44px (Level AAA) for optimal accessibility. Use CSS padding to increase touch area without affecting visual design."

      # Touch targets with insufficient padding
      - pattern: "padding:\\s*(?:0|1|2|3|4)\\s*(?:px|rem|em)"
        pattern_negate: "(min-width|min-height|max-width|max-height)"
        message: "Consider increasing padding to create larger touch targets. Google recommends 48x48px with 8px spacing, Apple recommends 44x44px minimum."

      # Interactive elements without proper touch target sizing
      - pattern: "<(button|a|input|select|textarea)[^>]*>"
        pattern_negate: "(min-width|min-height|width|height|padding)"
        message: "Interactive elements should have sufficient touch target size. Use CSS min-width/min-height or padding to ensure accessibility compliance."

      # Touch targets too close together (less than 8px spacing)
      - pattern: "(margin|gap):\\s*(?:0|1|2|3|4|5|6|7)\\s*(?:px|rem|em)"
        pattern_negate: "(min-width|min-height|max-width|max-height)"
        message: "Touch targets should have at least 8px spacing between them to prevent accidental activation. Consider increasing margins or gaps."

      # Fixed orientation restrictions
      - pattern: "orientation:\\s*(portrait|landscape)"
        message: "Avoid restricting content orientation. Allow users to consume content in any orientation they prefer or require for their computing environment."

      # Viewport meta tag preventing orientation changes
      - pattern: "<meta[^>]*name=\"viewport\"[^>]*>"
        pattern_negate: "(width=device-width|initial-scale)"
        message: "Viewport meta tag should support responsive design and orientation flexibility. Avoid restrictions that prevent content reflow."

      # CSS that prevents orientation flexibility
      - pattern: "@media\\s*\\(orientation:\\s*(portrait|landscape)\\)"
        pattern_negate: "(min-width|max-width|flex-direction|grid-template)"
        message: "Orientation media queries should enhance layout, not restrict functionality. Ensure content remains accessible in both orientations."

      # Fixed positioning that may cause issues in different orientations
      - pattern: "position:\\s*fixed"
        pattern_negate: "(top|bottom|left|right|transform)"
        message: "Fixed positioning may cause accessibility issues in different orientations. Consider using relative positioning with responsive adjustments."

      # Touch targets without proper focus indicators
      - pattern: "<(button|a|input|select|textarea)[^>]*>"
        pattern_negate: "(:focus|:focus-visible|outline|box-shadow)"
        message: "Touch targets should have visible focus indicators for keyboard navigation accessibility."

      # Mobile-specific elements without touch-friendly sizing
      - pattern: "(?i)<(button|a|input|select|textarea)[^>]*class=\"[^\"]*(?:mobile|touch|small)[^\"]*\"[^>]*>"
        pattern_negate: "(min-width|min-height|width|height|padding)"
        message: "Mobile-specific interactive elements must have adequate touch target sizes for accessibility compliance."

      # Grid layouts without proper spacing
      - pattern: "display:\\s*grid"
        pattern_negate: "(gap|grid-gap|margin|padding)"
        message: "Grid layouts should include proper spacing between elements to prevent accidental touch activation."

      # Flexbox layouts without proper spacing
      - pattern: "display:\\s*flex"
        pattern_negate: "(gap|margin|padding|justify-content|align-items)"
        message: "Flexbox layouts should include proper spacing and alignment to ensure touch target accessibility."

  - type: suggest
    message: |
      **WCAG Mobile Accessibility Requirements:**

      **Touch Target Size (WCAG 2.5.8 Target Size Minimum):**
      - **Level AA (Required):** Minimum 24x24 pixels
      - **Recommended:** 44x44 pixels or larger for optimal usability
      - **Industry Standards:** Google (48x48px), Apple (44x44px)
      - **Spacing:** At least 8 pixels between touch targets

      **Touch Target Implementation:**

      **1. CSS Padding Approach:**
      ```css
      /* Good: Touch target meeting minimum requirements */
      .touch-button {
        min-width: 24px;  /* WCAG 2.5.8 AA minimum */
        min-height: 24px;
        padding: 12px 16px; /* Padding increases effective touch area */
        border: none;
        background: #0056b3;
        color: white;
        border-radius: 4px;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Shopify/horizon](https://github.com/Shopify/horizon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->

---
trigger: always_on
description: Focus order and focus styles accessibility standards per WCAG 2.4.7 Focus Visible, 1.4.11 Non-Text Contrast, 2.4.13 Focus Appearance, and 2.4.11 Focus Not Obscured requirements
---


# Focus Order and Focus Styles Accessibility Standards

Ensures proper focus order, tabindex usage, and focus indicators following WCAG 2.4.7 Focus Visible, 1.4.11 Non-Text Contrast, 2.4.13 Focus Appearance, and 2.4.11 Focus Not Obscured requirements.

<rule>
name: focus_order_and_styles_accessibility_standards
description: Enforce focus order and focus styles accessibility standards per WCAG requirements
filters:
  - type: file_extension
    pattern: "\\.(vue|jsx|tsx|html|liquid|php|js|ts|css|scss|sass|less)$"

actions:
  - type: enforce
    conditions:
      # Positive tabindex values (should not be used)
      - pattern: "tabindex=\"[1-9]\""
        message: "Positive tabindex values create illogical focus order. Use DOM order instead or tabindex=\"0\" for custom focusable elements."

      # Missing focus styles (outline: 0 or outline: none)
      - pattern: "outline:\\s*0|outline:\\s*none"
        message: "Focus styles should not be removed. Use custom focus indicators that meet WCAG contrast requirements."

      # Focus styles with insufficient contrast (light colors)
      - pattern: "outline.*#[89abcdefABCDEF]{6}|outline.*#[cdefCDEF]{3,6}"
        message: "Light focus outline colors may not meet 3:1 contrast ratio requirement for UI component identification."

      # Missing focus-visible implementation
      - pattern: ":focus\\s*\\{"
        pattern_negate: ":focus-visible|:focus:not\\(:focus-visible\\)"
        message: "Consider implementing :focus-visible for better keyboard-only focus indication."

      # Focus styles that may be obscured
      - pattern: "outline-offset:\\s*-?0\\.?0*px|outline-offset:\\s*0"
        message: "Consider using positive outline-offset to prevent focus indicators from being obscured by adjacent elements."

      # Missing forced-colors media query for Windows High Contrast
      - pattern: "@media\\s*\\(forced-colors:\\s*active\\)"
        pattern_negate: "outline.*transparent"
        message: "Windows High Contrast Mode requires transparent outline for native focus appearance."

      # Custom focusable elements without proper tabindex
      - pattern: "<(div|span|button)[^>]*onclick|onkeydown|onkeypress"
        pattern_negate: "tabindex=\"[0-9]\"|role=\"button\"|role=\"link\""
        message: "Custom interactive elements should have tabindex=\"0\" or appropriate ARIA role for keyboard accessibility."

      # Focus styles with insufficient area
      - pattern: "outline-width:\\s*1px|outline-width:\\s*0\\.1rem"
        message: "Thin focus outlines may not meet WCAG 2.4.13 Focus Appearance requirements for minimum area."

      # Focus styles that blend with background
      - pattern: "outline.*rgba\\([^)]*0\\.1[^)]*\\)|outline.*rgba\\([^)]*0\\.2[^)]*\\)"
        message: "Very transparent focus outlines may not provide sufficient contrast for visibility."

      # Missing focus styles on interactive elements
      - pattern: "<(button|a|input|select|textarea)[^>]*>"
        pattern_negate: ":focus|:focus-visible|tabindex"
        message: "Interactive elements should have visible focus styles for keyboard navigation accessibility."

      # Dynamic content removal without focus management
      - pattern: "\\.remove\\(\\)|removeChild|innerHTML\\s*="
        pattern_negate: "focus\\(|focus\\(\\)"
        message: "When removing dynamic content, ensure proper focus management by restoring focus to a logical location."

  - type: suggest
    message: |
      **WCAG Focus Order and Focus Styles Requirements:**

      **Focus Order Requirements:**

      **1. Logical DOM Order:**
      - **Default:** Focus order follows DOM element order
      - **Navigation:** Tab key moves forward, Shift+Tab moves backward
      - **Avoid:** Positive tabindex values (1, 2, 3, etc.)

      **2. Tabindex Usage:**
      ```html
      <!-- Good: Use DOM order (default) -->
      <button>First Button</button>
      <button>Second Button</button>
      <button>Third Button</button>

      <!-- Good: tabindex="0" for custom focusable elements -->
      <div role="button" tabindex="0" onclick="handleClick()">
        Custom Button
      </div>

      <!-- Good: tabindex="-1" for programmatic focus only -->
      <div id="target" tabindex="-1">Focus target</div>
      <button onclick="document.getElementById('target').focus()">
        Focus Target
      </button>

      <!-- Bad: Positive tabindex values -->
      <button tabindex="1">First</button>
      <button tabindex="3">Third</button>
      <button tabindex="2">Second</button>
      ```

      **Focus Styles Requirements:**

      **1. WCAG 2.4.7 Focus Visible (Level A):**
      - **Requirement:** Focus indicator must exist
      - **Purpose:** Keyboard users need visible focus indication

      **2. WCAG 1.4.11 Non-Text Contrast (Level AA):**
      - **Requirement:** Minimum 3:1 contrast ratio for UI components
      - **Applies to:** Focus indicators, borders, focus outlines

      **3. WCAG 2.4.13 Focus Appearance (Level AAA):**
      - **Requirement:** Minimum area and contrast for focus indicators
      - **Area:** Focus indicator should be clearly visible


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Shopify/horizon](https://github.com/Shopify/horizon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->

---
trigger: always_on
description: Text and user interface color contrast compliance with WCAG 2.2 1.4.3 and 1.4.11
---

# Color Contrast Accessibility Standards

Ensures color contrast meets WCAG 2.2 1.4.3: Contrast (Minimum) and 1.4.11: Non-text Contrast requirements.

<rule>
name: color_contrast_accessibility_standards
description: Enforce color contrast accessibility standards per WCAG 2.2 1.4.3 and 1.4.11
filters:
  - type: file_extension
    pattern: "\\.(css|scss|sass|less|vue|jsx|tsx|html|liquid|php|js|ts)$"

actions:
  - type: enforce
    conditions:
      # Common low-contrast text color combinations
      - pattern: "color:\\s*#[89abcdefABCDEF]{6}"
        message: "Light text colors may not meet 4.5:1 contrast ratio requirement. Verify contrast against background."

      - pattern: "color:\\s*#[0-6]{6}"
        message: "Very light text colors likely fail contrast requirements. Use darker colors for better accessibility."

      # Light gray text (common accessibility issue)
      - pattern: "color:\\s*(#[cdefCDEF]{3,6}|lightgray|lightgrey|silver)"
        message: "Light gray text often fails WCAG contrast requirements (4.5:1 minimum). Use darker colors."

      # Common problematic color combinations
      - pattern: "background.*#fff.*color.*#[89abcdefABCDEF]"
        message: "Light text on white background may not meet 4.5:1 contrast ratio requirement."

      - pattern: "background.*#f[0-9a-fA-F]{5}.*color.*#[89abcdefABCDEF]"
        message: "Light text on light background may not meet contrast requirements."

      # UI component border/focus indicators
      - pattern: "border.*#[cdefCDEF]{3,6}"
        message: "Light borders may not meet 3:1 non-text contrast requirement for UI components."

      - pattern: "outline.*#[cdefCDEF]{3,6}"
        message: "Light focus outlines may not meet 3:1 contrast requirement for UI component identification."

      # Button states with insufficient contrast
      - pattern: "button.*background.*#[cdefCDEF]{3,6}"
        message: "Light button backgrounds may not provide sufficient 3:1 contrast for UI component identification."

      # Form input borders
      - pattern: "input.*border.*#[defDEF]{3,6}"
        message: "Very light input borders may not meet 3:1 contrast requirement for form field identification."

      # SVG icon fill colors that may lack contrast
      - pattern: "fill=\"#[cdefCDEF]{3,6}\""
        message: "Light SVG fill colors may not meet 3:1 contrast requirement for icon identification."

      # SVG stroke colors for icon outlines
      - pattern: "stroke=\"#[defDEF]{3,6}\""
        message: "Very light SVG stroke colors may not provide sufficient contrast for icon visibility."

      # Missing prefers-contrast considerations
      - pattern: "@media\\s*\\(prefers-contrast:\\s*more\\)"
        pattern_negate: "color|background|border"
        message: "prefers-contrast: more media query should include enhanced color/contrast properties."

  - type: suggest
    message: |
      **WCAG 2.2 Color Contrast Requirements:**

      **1.4.3: Text Contrast (Minimum) - Level AA:**
      - **Normal Text:** Minimum 4.5:1 contrast ratio
      - **Large Text:** Minimum 3:1 contrast ratio (18pt+ regular or 14pt+ bold)
      - **Enhanced (Level AAA):** 7:1 for normal text, 4.5:1 for large text

      **1.4.11: Non-text Contrast - Level AA:**
      - **UI Components:** Minimum 3:1 contrast ratio for component identification
      - **Focus Indicators:** Minimum 3:1 contrast ratio for focus visibility
      - **Graphical Objects:** Minimum 3:1 contrast ratio for content understanding

      **Exceptions (No Contrast Requirement):**
      - Inactive/disabled UI components
      - Pure decorative elements
      - Text in logos or brand names
      - Text that is not visible to users
      - Graphics where specific presentation is essential

      **High Contrast Color Combinations:**

      **Dark Text on Light Backgrounds:**
      - `#212529` on `#ffffff` - 16.6:1 ✅
      - `#495057` on `#ffffff` - 8.3:1 ✅
      - `#6c757d` on `#ffffff` - 5.4:1 ✅
      - `#343a40` on `#f8f9fa` - 11.7:1 ✅

      **Light Text on Dark Backgrounds:**
      - `#ffffff` on `#212529` - 16.6:1 ✅
      - `#f8f9fa` on `#495057` - 7.0:1 ✅
      - `#ffffff` on `#0056b3` - 7.7:1 ✅
      - `#ffffff` on `#dc3545` - 5.8:1 ✅

      **UI Component Colors (3:1 minimum):**
      - Focus outlines: `#0056b3`, `#dc3545`, `#198754`
      - Border colors: `#ced4da`, `#adb5bd`, `#6c757d`
      - Button states: `#0056b3`, `#157347`, `#b02a37`

      **Implementation Examples:**

      **CSS Text Contrast:**
      ```css
      /* Good: High contrast text */
      .primary-text {
        color: #212529;
        background: #ffffff;
      }

      .secondary-text {
        color: #495057;
        background: #ffffff;
      }

      /* Good: Large text with 3:1 minimum */
      .large-heading {
        font-size: 18px;
        font-weight: normal;
        color: #6c757d;
        background: #ffffff;
      }
      ```

      **CSS UI Component Contrast:**
      ```css
      /* Good: Form inputs with sufficient border contrast */
      .form-control {
        border: 2px solid #ced4da; /* 3:1+ contrast */
        background: #ffffff;
      }

      .form-control:focus {
        border-color: #0056b3; /* High contrast focus */
        outline: 3px solid #0056b3;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Shopify/horizon](https://github.com/Shopify/horizon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->

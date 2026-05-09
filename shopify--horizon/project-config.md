---
trigger: always_on
description: Dropdown Navigation component accessibility compliance pattern
---

# Dropdown Navigation Component Accessibility Standards

Ensures dropdown navigation components follow WCAG compliance and proper navigation semantics, including mobile modal patterns and disclosure controls.

<rule>
name: dropdown_navigation_accessibility_standards
description: Enforce dropdown navigation component accessibility standards and proper navigation semantics
filters:
  - type: file_extension
    pattern: "\\.(vue|jsx|tsx|html|liquid|php|js|ts)$"

actions:
  - type: enforce
    conditions:
      # Navigation landmark requirement
      - pattern: "(?i)<nav[^>]*(?:navigation|menu|dropdown)[^>]*>"
        pattern_negate: "(aria-label|aria-labelledby)=\"[^\"]+\""
        message: "Navigation elements must have aria-label or aria-labelledby attribute for accessibility."

      # Navigation list structure requirement
      - pattern: "(?i)<nav[^>]*(?:navigation|menu|dropdown)[^>]*>"
        pattern_negate: "<ul[^>]*>"
        message: "Navigation should use unordered list (ul) for proper semantic structure."

      # Dropdown button role requirement
      - pattern: "(?i)<(button|div|span)[^>]*(?:dropdown|expand|collapse)[^>]*>"
        pattern_negate: "role=\"button\""
        message: "Dropdown controls must have role='button' (or use native button element which has implicit role)."

      # Dropdown button aria-expanded requirement
      - pattern: "(?i)<[^>]*role=\"button\"[^>]*(?:dropdown|expand|collapse)[^>]*>"
        pattern_negate: "aria-expanded=\"(true|false)\""
        message: "Dropdown controls must have aria-expanded attribute set to 'true' or 'false'."

      # Dropdown content missing proper identification
      - pattern: "(?i)<(div|section)[^>]*(?:dropdown.*content|content.*dropdown)[^>]*>"
        pattern_negate: "id=\"[^\"]+\""
        message: "Dropdown content must have unique ID attributes for aria-controls reference."

      # Missing aria-current on navigation items
      - pattern: "(?i)<a[^>]*(?:nav|navigation)[^>]*>"
        pattern_negate: "aria-current=\"(page|false)\""
        message: "Navigation links should have aria-current attribute set to 'page' for active items or 'false' for inactive."

      # Mobile modal missing dialog role
      - pattern: "(?i)<(div|section)[^>]*(?:mobile.*nav|nav.*mobile|modal.*nav)[^>]*>"
        pattern_negate: "role=\"dialog\""
        message: "Mobile navigation modal containers must have role='dialog' attribute."

      # Mobile modal missing aria-modal
      - pattern: "(?i)<[^>]*role=\"dialog\"[^>]*(?:mobile.*nav|nav.*mobile)[^>]*>"
        pattern_negate: "aria-modal=\"true\""
        message: "Mobile navigation dialog elements must have aria-modal='true' attribute."

      # Mobile modal missing proper labeling
      - pattern: "(?i)<[^>]*role=\"dialog\"[^>]*(?:mobile.*nav|nav.*mobile)[^>]*>"
        pattern_negate: "(aria-labelledby|aria-label)"
        message: "Mobile navigation dialog elements must have either aria-labelledby or aria-label for accessibility."

      # Mobile launcher missing aria-haspopup
      - pattern: "(?i)<button[^>]*(?:mobile.*nav|nav.*mobile|hamburger|menu)[^>]*>"
        pattern_negate: "aria-haspopup=\"dialog\""
        message: "Mobile navigation launcher buttons must include aria-haspopup='dialog' to inform users a dialog will open."

      # Mobile close button missing aria-label
      - pattern: "(?i)<button[^>]*(?:close|dismiss|×|&times;)[^>]*(?:mobile.*nav|nav.*mobile)[^>]*>"
        pattern_negate: "aria-label=\"[^\"]*[Cc]lose[^\"]*\""
        message: "Mobile navigation close buttons should have aria-label='Close navigation' or similar descriptive text."

      # Missing keyboard event handlers for dropdown
      - pattern: "(?i)<[^>]*role=\"button\"[^>]*(?:dropdown|expand|collapse)[^>]*>"
        pattern_negate: "(onKeyDown|onkeydown|@keydown|v-on:keydown)"
        message: "Dropdown controls should handle keyboard events (Enter, Space, and Escape)."

      # Missing Escape key support for dropdown content
      - pattern: "(?i)<div[^>]*(?:dropdown.*content|content.*dropdown)[^>]*>"
        pattern_negate: "(onKeyDown|onkeydown|@keydown|v-on:keydown)"
        message: "Dropdown content areas should handle Escape key to close dropdown and return focus to launcher."

      # Incorrect menu role usage
      - pattern: "(?i)role=\"(menu|menuitem|menubar|menuitemcheckbox|menuitemradio)\""
        message: "Navigation components should NOT use menu roles. Use proper navigation semantics with ul/li/a elements."

      # Incorrect aria-haspopup usage
      - pattern: "(?i)aria-haspopup=\"(true|menu|listbox)\""
        pattern_negate: "aria-haspopup=\"dialog\""
        message: "Navigation components should NOT use aria-haspopup except for mobile modal launchers with aria-haspopup='dialog'."

  - type: suggest
    message: |
      **Dropdown Navigation Component Accessibility Best Practices:**

      **Navigation Semantics:**
      - **role='navigation':** Implicit on nav element, provides landmark
      - **aria-label/aria-labelledby:** On nav element to describe the navigation
      - **aria-current:** Set on active navigation items ('page' for current page, 'false' for inactive)
      - **ul + li + a:** Use semantic list structure for navigation items

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Shopify/horizon](https://github.com/Shopify/horizon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->

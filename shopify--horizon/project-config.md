---
trigger: always_on
description: Combobox component accessibility compliance pattern
---

# Combobox Component Accessibility Standards

Ensures combobox components follow WCAG compliance and WAI-ARIA Combobox Pattern specifications.

<rule>
name: combobox_accessibility_standards
description: Enforce combobox component accessibility standards and WAI-ARIA Combobox Pattern compliance
filters:
  - type: file_extension
    pattern: "\\.(vue|jsx|tsx|html|liquid|php|js|ts)$"

actions:
  - type: enforce
    conditions:
      # Combobox role requirement
      - pattern: "(?i)<(div|section)[^>]*(?:combobox|autocomplete)[^>]*>"
        pattern_negate: "role=\"combobox\""
        message: "Combobox containers must have role='combobox' attribute."

      # aria-expanded requirement
      - pattern: "(?i)<[^>]*role=\"combobox\"[^>]*>"
        pattern_negate: "aria-expanded=\"(true|false)\""
        message: "Combobox elements must have aria-expanded attribute set to 'true' or 'false'."

      # aria-haspopup requirement
      - pattern: "(?i)<[^>]*role=\"combobox\"[^>]*>"
        pattern_negate: "aria-haspopup=\"listbox\""
        message: "Combobox elements must have aria-haspopup='listbox' attribute."

      # aria-controls requirement
      - pattern: "(?i)<[^>]*role=\"combobox\"[^>]*>"
        pattern_negate: "aria-controls=\"[^\"]+\""
        message: "Combobox elements must have aria-controls attribute referencing the ID of the associated listbox."

      # aria-autocomplete requirement
      - pattern: "(?i)<[^>]*role=\"combobox\"[^>]*>"
        pattern_negate: "aria-autocomplete=\"(list|both|inline|none)\""
        message: "Combobox elements must have aria-autocomplete attribute set to 'list', 'both', 'inline', or 'none'."

      # aria-activedescendant requirement when expanded
      - pattern: "(?i)<[^>]*role=\"combobox\"[^>]*aria-expanded=\"true\"[^>]*>"
        pattern_negate: "aria-activedescendant=\"[^\"]+\""
        message: "Expanded combobox elements must have aria-activedescendant attribute referencing the ID of the active option."

      # Listbox role requirement
      - pattern: "(?i)<(div|ul)[^>]*(?:listbox|dropdown|popup)[^>]*>"
        pattern_negate: "role=\"listbox\""
        message: "Listbox containers must have role='listbox' attribute."

      # Option role requirement
      - pattern: "(?i)<(div|li)[^>]*(?:option|item)[^>]*>"
        pattern_negate: "role=\"option\""
        message: "Listbox options must have role='option' attribute."

      # Option ID requirement for aria-activedescendant
      - pattern: "(?i)<[^>]*role=\"option\"[^>]*>"
        pattern_negate: "id=\"[^\"]+\""
        message: "Listbox options must have unique id attributes for aria-activedescendant to reference them."

      # aria-selected requirement for options
      - pattern: "(?i)<[^>]*role=\"option\"[^>]*>"
        pattern_negate: "aria-selected=\"(true|false)\""
        message: "Listbox options must have aria-selected attribute set to 'true' or 'false'."

      # Missing keyboard event handlers
      - pattern: "(?i)<[^>]*role=\"combobox\"[^>]*>"
        pattern_negate: "(onKeyDown|onkeydown|@keydown|v-on:keydown)"
        message: "Combobox elements should handle keyboard events (Arrow keys, Enter, Escape, etc.)."

      # Missing status region
      - pattern: "(?i)<[^>]*role=\"combobox\"[^>]*>"
        pattern_negate: "aria-controls=\"[^\"]+\".*?<[^>]*role=\"status\""
        message: "Combobox should have a status region to announce available options."

  - type: suggest
    message: |
      **Combobox Component Accessibility Best Practices:**

      **Required ARIA Attributes:**
      - **role='combobox':** Set on the input container element
      - **aria-expanded:** 'true' if listbox is visible, 'false' if hidden
      - **aria-haspopup='listbox':** Indicates the combobox has a listbox popup
      - **aria-controls:** Reference to the ID of the associated listbox
      - **aria-autocomplete:** 'list', 'both', 'inline', or 'none' based on behavior
      - **aria-activedescendant:** Reference to the ID of the currently active option (remove when listbox is hidden)
      - **role='listbox':** Set on the popup container element (preferably on a `ul` element)
      - **role='option':** Set on each selectable item in the listbox (preferably on an `li` element)
      - **id:** Unique ID on each option element for `aria-activedescendant` to reference
      - **aria-selected:** 'true' or 'false' on each option
      - **role='status':** Set on a visually hidden element to announce available options

      **Keyboard Interaction Requirements:**
      - **Down Arrow:** Open listbox and move focus to first option
      - **Up Arrow:** Open listbox and move focus to last option
      - **Enter/Space:** Select focused option and close listbox
      - **Escape:** Close listbox without selection
      - **Tab:** Move focus to next focusable element
      - **Shift+Tab:** Move focus to previous focusable element
      - **Home/End:** Move focus to first/last option
      - **Character Keys:** Filter options based on input

      **Focus Management:**
      - Focus should remain on the input while navigating options
      - Use aria-activedescendant to indicate the currently focused option
      - Return focus to input after selection or closing
      - Ensure focus is trapped within the combobox while open

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Shopify/horizon](https://github.com/Shopify/horizon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->

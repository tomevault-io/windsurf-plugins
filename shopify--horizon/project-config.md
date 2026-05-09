---
trigger: always_on
description: Product filter component accessibility compliance pattern
---


# Product Filter Component Accessibility Standards

Ensures product filter components follow WCAG compliance and WAI-ARIA Disclosure Pattern specifications, including sort controls and grid layout buttons.

<rule>
name: product_filter_accessibility_standards
description: Enforce product filter component accessibility standards and WAI-ARIA Disclosure Pattern compliance
filters:
  - type: file_extension
    pattern: "\\.(vue|jsx|tsx|html|liquid|php|js|ts)$"

actions:
  - type: enforce
    conditions:
      # Filter disclosure button role requirement (for non-button elements)
      - pattern: "(?i)<(div|span)[^>]*(?:filter|disclosure|expand|collapse)[^>]*>"
        pattern_negate: "role=\"button\""
        message: "Non-button filter disclosure controls must have role='button'. Native button elements have implicit role and don't need explicit role attribute."

      # Filter disclosure aria-expanded requirement
      - pattern: "(?i)<[^>]*role=\"button\"[^>]*(?:filter|disclosure|expand|collapse)[^>]*>"
        pattern_negate: "aria-expanded=\"(true|false)\""
        message: "Filter disclosure controls must have aria-expanded attribute set to 'true' or 'false'."

      # Filter disclosure missing keyboard event handlers
      - pattern: "(?i)<[^>]*role=\"button\"[^>]*(?:filter|disclosure|expand|collapse)[^>]*>"
        pattern_negate: "(onKeyDown|onkeydown|@keydown|v-on:keydown)"
        message: "Filter disclosure controls should handle keyboard events (Enter, Space, and Escape)."

      # Missing Escape key support for filter content
      - pattern: "(?i)<div[^>]*(?:filter.*content|content.*filter)[^>]*>"
        pattern_negate: "(onKeyDown|onkeydown|@keydown|v-on:keydown)"
        message: "Filter content areas should handle Escape key to close filter and return focus to launcher."

      # Filter disclosure content not a sibling
      - pattern: "(?i)<[^>]*role=\"button\"[^>]*(?:filter|disclosure)[^>]*>"
        pattern_negate: "<(div|section)[^>]*id=\"[^\"]+\"[^>]*>"
        message: "Filter disclosure content must be a sibling to the disclosure control in the DOM."

      # Grid layout buttons missing aria-current
      - pattern: "(?i)<(button|div|span)[^>]*(?:grid|layout|view)[^>]*>"
        pattern_negate: "aria-current=\"(true|false)\""
        message: "Grid layout buttons must have aria-current attribute set to 'true' or 'false'."

      # Sort filter missing proper labeling
      - pattern: "(?i)<(button|div|span)[^>]*(?:sort|order)[^>]*>"
        pattern_negate: "(aria-label|aria-labelledby)"
        message: "Sort filter controls should have proper labeling for screen reader context."

      # Sort filter using checkboxes instead of radio buttons
      - pattern: "(?i)<input[^>]*type=\"checkbox\"[^>]*(?:sort|order)[^>]*>"
        message: "Sort filter options should use radio buttons since only one option can be selected at a time."

      # Checkbox groups missing fieldset
      - pattern: "(?i)<input[^>]*type=\"checkbox\"[^>]*(?:filter|option)[^>]*>"
        pattern_negate: "<fieldset"
        message: "Filter checkbox groups should be wrapped in fieldset elements for proper grouping."

      # Fieldset missing legend
      - pattern: "(?i)<fieldset[^>]*(?:filter|option)[^>]*>"
        pattern_negate: "<legend"
        message: "Filter fieldsets must have legend elements to provide context for the group."

      # Filter options missing proper IDs
      - pattern: "(?i)<input[^>]*type=\"checkbox\"[^>]*(?:filter|option)[^>]*>"
        pattern_negate: "id=\"[^\"]+\""
        message: "Filter checkboxes should have unique ID attributes for proper labeling."

      # Missing product count live region
      - pattern: "(?i)<[^>]*(?:product.*count|count.*product)[^>]*>"
        pattern_negate: "role=\"status\""
        message: "Product count displays should use role='status' for screen reader announcements."



      # Missing main products heading
      - pattern: "(?i)<[^>]*(?:product.*filter|filter.*product)[^>]*>"
        pattern_negate: "<h1[^>]*>.*[Pp]roducts?[^<]*</h1>"
        message: "Product filter pages should have an h1 heading with 'Products' for proper page structure."

  - type: suggest
    message: |
      **Product Filter Component Accessibility Best Practices:**

      **Page Structure Requirements:**
      - Use `<h1>` for the main "Products" heading
      - Wrap filter controls and product count in a `<div class="products-header">`
      - Remove separate section headings for filters and product cards
      - Present filters, count, and products as one cohesive section

      **Product Count Live Region:**
      - Add product count display with `role="status"`
      - Use unique ID for the count text element (e.g., `id="product-count-text"`)
      - Update count dynamically as filters are applied/removed
      - Ensure count is announced to screen readers when it changes

      **Required ARIA Attributes:**
      - **role='button':** Only required for non-button elements (native button elements have implicit role)
      - **aria-expanded:** 'true' if filter content is visible, 'false' if hidden
      - **aria-controls:** Reference to the ID of the associated filter content
      - **aria-current:** Set on grid layout buttons ('true' for active, 'false' for inactive)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Shopify/horizon](https://github.com/Shopify/horizon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->

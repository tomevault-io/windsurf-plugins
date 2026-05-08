---
trigger: always_on
description: Cart drawer component accessibility compliance pattern
---

# Cart Drawer Component Accessibility Standards

Ensures cart drawer components follow WCAG compliance and ARIA Dialog Pattern specifications for ecommerce applications.

<rule>
name: cart_drawer_accessibility_standards
description: Enforce cart drawer component accessibility standards and ARIA Dialog Pattern compliance
filters:
  - type: file_extension
    pattern: "\\.(vue|jsx|tsx|html|liquid|php|js|ts)$"

actions:
  - type: enforce
    conditions:
      # Cart activator missing aria-haspopup
      - pattern: "(?i)<button[^>]*(?:cart|basket|shopping)[^>]*>"
        pattern_negate: "aria-haspopup=\"dialog\""
        message: "Cart activator buttons must include aria-haspopup='dialog' to inform users a dialog will open."

      # Cart container missing dialog role
      - pattern: "(?i)<(div|section|aside)[^>]*(?:cart|basket|drawer)[^>]*>"
        pattern_negate: "role=\"dialog\""
        message: "Cart drawer containers must have role='dialog' attribute."

      # Cart container missing aria-modal
      - pattern: "(?i)<[^>]*role=\"dialog\"[^>]*(?:cart|basket|drawer)[^>]*>"
        pattern_negate: "aria-modal=\"true\""
        message: "Cart drawer dialog elements must have aria-modal='true' attribute."

      # Cart container missing proper labeling
      - pattern: "(?i)<[^>]*role=\"dialog\"[^>]*(?:cart|basket|drawer)[^>]*>"
        pattern_negate: "(aria-labelledby|aria-label)"
        message: "Cart drawer dialog elements must have either aria-labelledby or aria-label for accessibility."

      # Empty aria-label check
      - pattern: "(?i)<[^>]*role=\"dialog\"[^>]*(?:cart|basket|drawer)[^>]*aria-label=\"\"[^>]*>"
        message: "Cart drawer aria-label should not be empty; provide a meaningful description like 'Shopping Cart'."

      # Close button missing proper functionality
      - pattern: "(?i)<button[^>]*(?:close|dismiss|cancel)[^>]*(?:cart|basket|drawer)[^>]*>"
        pattern_negate: "(onClick|onclick|@click|v-on:click)"
        message: "Cart drawer close buttons should have proper click handlers to close the dialog."

      # Close button missing aria-label
      - pattern: "(?i)<button[^>]*(?:close|dismiss|×|&times;)[^>]*(?:cart|basket|drawer)[^>]*>"
        pattern_negate: "aria-label=\"[^\"]*[Cc]lose[^\"]*\""
        message: "Cart drawer close buttons should have aria-label='Close cart' or similar descriptive text."

      # Missing focus management indicators
      - pattern: "(?i)(?:openCart|showCart|toggleCart|openDrawer)\\s*\\("
        message: "When opening cart drawers, ensure focus management is implemented (focus should move to first focusable element inside the dialog)."

      # Missing checkout button accessibility
      - pattern: "(?i)<button[^>]*(?:checkout|proceed|purchase)[^>]*(?:cart|basket|drawer)[^>]*>"
        pattern_negate: "(aria-label|aria-describedby)"
        message: "Cart drawer checkout buttons should have proper labeling for screen readers."

      # Quantity inputs missing aria-live for screen reader announcements
      - pattern: "(?i)<input[^>]*type=\"number\"[^>]*(?:quantity|qty)[^>]*>"
        pattern_negate: "aria-live=\"polite\""
        message: "Cart quantity inputs must have aria-live='polite' to announce value changes to screen readers."

      # Missing focus management for item removal
      - pattern: "(?i)(?:removeItem|remove.*item|delete.*item)\\s*\\("
        pattern_negate: "focus\\(|focus\\(\\).*close|close.*focus\\(\\"
        message: "When removing cart items, implement focus management to shift focus to a logical location (e.g., close button) for better user experience."

  - type: suggest
    message: |
      **Cart Drawer Component Accessibility Best Practices:**

      **Required ARIA Attributes:**
      - **aria-haspopup='dialog':** Set on cart activator buttons to inform users a dialog will open
      - **role='dialog':** Set on the cart drawer container element
      - **aria-modal='true':** Indicates the cart drawer is modal and traps focus
      - **aria-labelledby:** Reference to visible cart title, OR
      - **aria-label:** Descriptive label like "Shopping Cart" if no visible title exists

      **Keyboard Interaction Requirements:**
      - **Initial Focus:** When cart drawer opens, focus must move to the first focusable element (typically close button)
      - **Tab Cycling:** Tab key should cycle through focusable elements within the cart drawer only
      - **Shift+Tab:** Should cycle backwards through focusable elements within the cart drawer
      - **Escape Key:** Must close the cart drawer and return focus to the activator
      - **Focus Trap:** Focus should be contained within the cart drawer while open

      **Focus Management:**
      - Implement focus trapping to prevent tab navigation outside the cart drawer
      - Return focus to the cart activator when drawer closes
      - Move focus to the close button (first focusable element) when drawer opens
      - Ensure close button is positioned first in DOM order within the dialog container
      - **Item Removal Focus:** When removing cart items, shift focus to the close button for logical positioning
      - **Quantity Changes:** Maintain focus on quantity controls during updates to prevent focus loss

      **Screen Reader Interaction:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Shopify/horizon](https://github.com/Shopify/horizon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->

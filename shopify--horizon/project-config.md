---
trigger: always_on
description: Flip Card component accessibility compliance pattern
---

# Flip Card Component Accessibility Standards

Ensures flip card components follow WCAG compliance and provide proper state management for screen reader users.

<rule>
name: flip_card_accessibility_standards
description: Enforce flip card component accessibility standards and proper state management
filters:
  - type: file_extension
    pattern: "\\.(vue|jsx|tsx|html|liquid|php|js|ts)$"

actions:
  - type: enforce
    conditions:
      # Flip button requirement
      - pattern: "(?i)<(div|section)[^>]*(?:card|flip)[^>]*>"
        pattern_negate: "<button[^>]*aria-pressed"
        message: "Flip cards must contain a button with aria-pressed attribute to control card state."

      # aria-pressed attribute requirement
      - pattern: "(?i)<button[^>]*(?:flip|card)[^>]*>"
        pattern_negate: "aria-pressed=\"(true|false)\""
        message: "Flip card buttons must have aria-pressed attribute set to 'true' or 'false'."

      # Card front/back structure requirement
      - pattern: "(?i)<(div|section)[^>]*(?:card|flip)[^>]*>"
        pattern_negate: "(card--front|card--back|front|back)"
        message: "Flip cards must have both front and back content sections for proper structure."

      # Unique accessible name requirement
      - pattern: "(?i)<button[^>]*aria-pressed[^>]*>"
        pattern_negate: "(aria-label|aria-labelledby|>.*[A-Za-z]{10,})"
        message: "Flip card buttons must have unique, descriptive accessible names that reference visible card content."

      # Keyboard focus indicator requirement
      - pattern: "(?i)<(div|section)[^>]*(?:card|flip)[^>]*>"
        pattern_negate: "(focus|:focus|focus-visible|:focus-visible)"
        message: "Flip card containers should have visible keyboard focus indicators when the flip button is focused."

      # Content visibility management
      - pattern: "(?i)aria-pressed=\"(true|false)\""
        pattern_negate: "(visibility.*hidden|display.*none|hidden)"
        message: "Use aria-pressed state to control content visibility - false shows front, true shows back. Prefer visibility: hidden/visible for smooth animations."

      # Missing flip button type
      - pattern: "(?i)<button[^>]*(?:flip|card)[^>]*>"
        pattern_negate: "type=\"button\""
        message: "Flip card buttons should have type='button' to prevent form submission behavior."

      # Incomplete card structure
      - pattern: "(?i)<div[^>]*class=\"card[^>]*>"
        pattern_negate: "(card--front.*card--back|card--back.*card--front)"
        message: "Flip cards must contain both front and back content sections for proper functionality."

  - type: suggest
    message: |
      **Flip Card Component Accessibility Best Practices:**

      **Required ARIA Attributes:**
      - **aria-pressed:** 'false' shows front content, 'true' shows back content
      - **type="button":** Prevents form submission behavior
      - **Unique accessible name:** Should reference visible card content

      **DOM Structure Requirements:**
      - Card container with front and back content sections
      - Flip button positioned between or adjacent to content sections
      - Use CSS display: none to hide non-visible content
      - Maintain logical reading order in the DOM

      **Content Visibility Management:**
      - **aria-pressed="false":** Show front content, hide back content
      - **aria-pressed="true":** Show back content, hide front content
      - Use CSS display property for smooth transitions
      - Ensure only one side is visible at a time

      **Keyboard and Focus Requirements:**
      - **Enter:** Toggle card state
      - **Space:** Toggle card state
      - **Tab:** Move focus to next focusable element
      - **Shift+Tab:** Move focus to previous focusable element
      - **Focus indicator:** Should wrap the card content container
      - **Hover state:** Blue border matching focus indicator for visual consistency

      **Implementation Example:**
      ```html
      <!-- ✅ Correct: Proper flip card structure -->
      <div class="card">
        <div class="card--front">
          <h3>Card Title</h3>
          <img src="front-image.jpg" alt="Front view of the product">
        </div>

        <button type="button"
                class="flip-button"
                aria-pressed="false"
                aria-label="More about Card Title">
        </button>

        <div class="card--back">
          <p class="card--tagline">Inspiring content</p>
          <img src="back-image-1.jpg" alt="Product detail view 1">
          <img src="back-image-2.jpg" alt="Product detail view 2">
          <img src="back-image-3.jpg" alt="Product detail view 3">
          <p>More detailed content about the product</p>
          <a href="/product-details">
            <img src="link-icon.svg" alt="View full product details">
          </a>
        </div>
      </div>
      ```

      **CSS Implementation:**
      ```css
      .card {
        position: relative;
        perspective: 1000px;
        /* Focus indicator for keyboard navigation */
        outline: 2px solid transparent;
        outline-offset: 2px;
        /* Visual affordance for clickable card */
        cursor: pointer;
      }

      .card:focus-within {
        outline-color: #0056b3;
        outline-width: 3px;
      }


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Shopify/horizon](https://github.com/Shopify/horizon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->

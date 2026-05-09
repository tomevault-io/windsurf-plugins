---
trigger: always_on
description: Sale price component accessibility compliance pattern
---

# Sale Price Component Accessibility Standards

Ensures sale price components follow WCAG compliance and provide proper context for screen reader users.

<rule>
name: sale_price_accessibility_standards
description: Enforce sale price component accessibility standards and screen reader context compliance
filters:
  - type: file_extension
    pattern: "\\.(vue|jsx|tsx|html|liquid|php|js|ts)$"

actions:
  - type: enforce
    conditions:
      # Sale price missing screen reader context
      - pattern: "(?i)<[^>]*(?:sale|discount|price)[^>]*>.*\\$[0-9]+.*\\$[0-9]+"
        pattern_negate: "(class.*sr-only|class.*visually-hidden)"
        message: "Sale price components must include visually hidden text to explain regular and sale prices."

      # Strike-through price missing context
      - pattern: "(?i)<[^>]*(?:strike|line-through|text-decoration)[^>]*>.*\\$[0-9]+"
        pattern_negate: "(Regular price|Original price|Was|Before)"
        message: "Strike-through prices must include visually hidden context like 'Regular price' or 'Original price'."

      # Sale price missing context
      - pattern: "(?i)<[^>]*(?:sale|discount|offer)[^>]*>.*\\$[0-9]+"
        pattern_negate: "(Sale price|Now|Current price|Discounted price)"
        message: "Sale prices must include visually hidden context like 'Sale price' or 'Now'."

      # Missing visually hidden elements for context
      - pattern: "(?i)<[^>]*class=\"[^\"]*(?:sale|price|discount)[^\"]*\"[^>]*>"
        pattern_negate: "(sr-only|visually-hidden|screen-reader-only)"
        message: "Sale price components should include visually hidden elements for screen reader context."

      # Unnecessary aria-hidden on visible price content
      - pattern: "(?i)<[^>]*class=\"[^\"]*(?:price|sale|discount)[^\"]*\"[^>]*aria-hidden=\"true\"[^>]*>"
        message: "Visible price content should not be hidden from screen readers with aria-hidden. Only decorative elements should use aria-hidden."

  - type: suggest
    message: |
      **Sale Price Component Accessibility Best Practices:**

      **Required Screen Reader Context:**
      - **Regular Price:** Must include "Regular price" or "Original price" context
      - **Sale Price:** Must include "Sale price" or "Now" context
      - **Visual Hiding:** Use visually hidden elements to provide context without affecting visual design

      **Screen Reader Announcement Requirements:**
      - Regular price should announce: "Regular price, $X"
      - Sale price should announce: "Sale price, $Y"
      - Discount should announce: "X% OFF" (visible badge text is sufficient)
      - Complete announcement: "Regular price, $45, Sale price, $35, 20% OFF"

      **Implementation Patterns:**

      **Basic Sale Price Structure:**
      ```html
      <div class="price-container">
        <span class="sr-only">Regular price, </span>
        <span class="regular-price">$45.00</span>
        <span class="sr-only">Sale price, </span>
        <span class="sale-price">$35.00</span>
        <span class="discount-badge">20% OFF</span>
      </div>
      ```

      **CSS for Visually Hidden Elements:**
      ```css
      .sr-only {
        position: absolute;
        width: 1px;
        height: 1px;
        padding: 0;
        margin: -1px;
        overflow: hidden;
        clip: rect(0, 0, 0, 0);
        white-space: nowrap;
        border: 0;
      }
      ```

      **Complex Sale Price Examples:**

      **With Multiple Discounts:**
      ```html
      <div class="price-container">
        <span class="sr-only">Regular price, </span>
        <span class="regular-price">$100.00</span>
        <span class="sr-only">Sale price, </span>
        <span class="sale-price">$75.00</span>
        <span class="discount">25% OFF</span>
        <span class="sr-only">Member price, </span>
        <span class="member-price">$67.50</span>
        <span class="savings-amount">Extra 10% OFF</span>
      </div>
      ```

      **With Currency and Formatting:**
      ```html
      <div class="price-container">
        <span class="sr-only">Regular price, </span>
        <span class="regular-price">$1,299.99</span>
        <span class="sr-only">Sale price, </span>
        <span class="sale-price">$999.99</span>
        <span class="savings-amount">Save $300</span>
      </div>
      ```

      **Button Implementation:**
      ```html
      <!-- Good: Natural button text -->
      <button class="add-to-cart-btn">
        Add to Cart
      </button>
      ```

      **JavaScript Considerations:**
      - Dynamically update visually hidden context when prices change
      - Calculate and update discount percentages automatically
      - Handle currency formatting for different locales
      - Ensure context is updated when prices are updated via AJAX

      **Dynamic Price Updates:**
      ```javascript
      function updateSalePrice(regularPrice, salePrice) {
        const regularElement = document.querySelector('.regular-price');
        const saleElement = document.querySelector('.sale-price');
        const discountElement = document.querySelector('.discount');

        // Calculate discount percentage
        const discount = Math.round(((regularPrice - salePrice) / regularPrice) * 100);

        // Update visual elements

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Shopify/horizon](https://github.com/Shopify/horizon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->

---
trigger: always_on
description: Product card accessibility compliance pattern
---

# Product Card Component Accessibility Standards

Ensures product card components follow WCAG compliance and implement proper single tab-stop navigation for keyboard and screen reader users.

<rule>
name: product_card_accessibility_standards
description: Enforce product card component accessibility standards and single tab-stop navigation compliance
filters:
  - type: file_extension
    pattern: "\\.(vue|jsx|tsx|html|liquid|php|js|ts)$"

actions:
  - type: enforce
    conditions:
      # Product card missing article wrapper
      - pattern: "(?i)<(div|section)[^>]*(?:product.*card|card.*product)[^>]*>"
        pattern_negate: "<article"
        message: "Product cards should be wrapped with the article element for semantic structure."

      # Product card missing proper link structure
      - pattern: "(?i)<article[^>]*(?:product.*card|card.*product)[^>]*>"
        pattern_negate: "<a[^>]*>.*?</a>"
        message: "Product cards must contain a link element for keyboard navigation and screen reader accessibility."

      # Product title not wrapped in link
      - pattern: "(?i)<(h1|h2|h3|h4|h5|h6)[^>]*(?:product.*title|title.*product)[^>]*>"
        pattern_negate: "<a[^>]*>.*?</a>"
        message: "Product titles should be wrapped in link elements to provide proper navigation context."

      # Heading not wrapping link element
      - pattern: "(?i)<(h1|h2|h3|h4|h5|h6)[^>]*(?:product.*title|title.*product)[^>]*>"
        pattern_negate: "<a[^>]*>"
        message: "Product card headings should wrap link elements to maintain proper heading semantics."

      # Missing product image alt text
      - pattern: "(?i)<img[^>]*(?:product|card)[^>]*>"
        pattern_negate: "alt=\"[^\"]+\""
        message: "Product card images must have descriptive alt text for screen reader users."

      # Empty alt text on product images
      - pattern: "(?i)<img[^>]*(?:product|card)[^>]*alt=\"\"[^>]*>"
        message: "Product card images should not have empty alt text; provide descriptive text or use alt=\"\" only for decorative images."

      # Product price missing proper semantic structure
      - pattern: "(?i)<(div|span)[^>]*(?:price|cost)[^>]*>"
        pattern_negate: "(<span[^>]*aria-label|aria-label=\"[^\"]*price[^\"]*\")"
        message: "Product prices should have proper semantic labeling for screen readers."

      # Product description not in paragraph element
      - pattern: "(?i)<(div|span)[^>]*(?:product.*description|description.*product)[^>]*>"
        pattern_negate: "<p"
        message: "Product descriptions should be wrapped in paragraph elements for proper semantic structure."

      # Missing focus indicators
      - pattern: "(?i)<a[^>]*(?:product.*card|card.*product)[^>]*>"
        pattern_negate: "(focus|hover|active)"
        message: "Product card links should have visible focus indicators for keyboard navigation."

      # Product card without proper positioning context
      - pattern: "(?i)<article[^>]*(?:product.*card|card.*product)[^>]*>"
        pattern_negate: "(position.*relative|position: relative)"
        message: "Product card containers should have position: relative for proper link overlay positioning."

      # Product card missing aria-labelledby
      - pattern: "(?i)<article[^>]*(?:product.*card|card.*product)[^>]*>"
        pattern_negate: "aria-labelledby=\"[^\"]+\""
        message: "Product card articles should have aria-labelledby referencing the heading ID for better screen reader context."

      # Product card heading missing ID
      - pattern: "(?i)<(h1|h2|h3|h4|h5|h6)[^>]*(?:product.*title|title.*product)[^>]*>"
        pattern_negate: "id=\"[^\"]+\""
        message: "Product card headings should have unique ID attributes for aria-labelledby reference."

      # Mouse-only link missing tabindex="-1"
      - pattern: "(?i)<a[^>]*class=\"[^\"]*product-link-mouse[^\"]*\"[^>]*>"
        pattern_negate: "tabindex=\"-1\""
        message: "Mouse-only product links should have tabindex='-1' to remove them from tab order."

  - type: suggest
    message: |
      **Product Card Component Accessibility Best Practices:**

      **Required Structure:**
      - **article element:** Wrap each product card with the article element for semantic meaning
      - **Single tab-stop:** Each product card should contain only one keyboard tab-stop
      - **Link overlay:** Use absolutely positioned link that covers the entire card area
      - **Heading wraps link:** The heading should wrap the link element to maintain proper heading semantics

      **ARIA and Semantic Requirements:**
      - **article role:** Implicit with article element, provides semantic structure
      - **aria-labelledby:** Reference to the heading ID for article labeling
      - **Heading ID:** Unique ID attribute for aria-labelledby reference
      - **Link text:** Product title should be descriptive and unique
      - **Image alt text:** Provide descriptive alt text for product images
      - **Price text:** Use visible text for pricing information
      - **Description text:** Use paragraph elements for product descriptions

      **Keyboard Navigation Requirements:**
      - **Single tab-stop:** Each product card should be navigable with one tab key press

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Shopify/horizon](https://github.com/Shopify/horizon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->

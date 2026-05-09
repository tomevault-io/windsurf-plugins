---
trigger: always_on
description: Global scope accessibility standards per WCAG requirements for page language, viewport, title attributes, and skip links
---


# Global Scope Accessibility Standards

Ensures global accessibility best practices are followed including page language, viewport meta tag, title attribute usage, and skip link implementation for WCAG compliance.

<rule>
name: global_accessibility_standards
description: Enforce global scope accessibility standards per WCAG requirements for page language, viewport, title attributes, and skip links
filters:
  - type: file_extension
    pattern: "\\.(vue|jsx|tsx|html|liquid|php|js|ts)$"

actions:
  - type: enforce
    conditions:
      # Missing lang attribute on html element
      - pattern: "<html[^>]*>"
        pattern_negate: "lang=\"[a-z]{2}(?:-[A-Z]{2})?\""
        message: "HTML element must have lang attribute set to ensure proper pronunciation by assistive technology. Use lang=\"en\" for English or appropriate language code."

      # Invalid language code format
      - pattern: "lang=\"[^a-z]{2}(?:-[^A-Z]{2})?\""
        message: "Language code must follow ISO 639-1 format (e.g., lang=\"en\", lang=\"fr-CA\")."

      # Missing viewport meta tag
      - pattern: "<head[^>]*>"
        pattern_negate: "<meta[^>]*name=\"viewport\""
        message: "Viewport meta tag is required for responsive design and accessibility. Include <meta name=\"viewport\" content=\"width=device-width, initial-scale=1.0\">."

      # Viewport meta tag preventing zoom
      - pattern: "<meta[^>]*name=\"viewport\"[^>]*>"
        pattern_negate: "(maximum-scale=1\\.0|user-scalable=no)"
        message: "Viewport meta tag should not prevent zooming. Avoid maximum-scale=1.0 and user-scalable=no to maintain accessibility for low-vision users."

      # Problematic viewport attributes
      - pattern: "(maximum-scale=1\\.0|user-scalable=no)"
        message: "Viewport meta tag contains attributes that prevent zooming, which creates accessibility issues for low-vision users. Remove maximum-scale=1.0 and user-scalable=no."

      # Title attribute on non-iframe elements
      - pattern: "(?i)<[^>]*title=\"[^\"]+\"[^>]*>"
        pattern_negate: "(iframe|IFRAME)"
        message: "Avoid using title attribute on non-iframe elements as it creates accessibility issues. Use visible text, aria-label, or custom tooltips instead."

      # Missing title on iframe elements
      - pattern: "<iframe[^>]*>"
        pattern_negate: "title=\"[^\"]+\""
        message: "Iframe elements must have title attribute to provide context for screen reader users."

      # Empty or meaningless iframe title
      - pattern: "<iframe[^>]*title=\"\\s*(?:iframe|frame|content|page)\\s*\"[^>]*>"
        message: "Iframe title should be descriptive and meaningful, not generic terms like 'iframe' or 'content'."

      # Missing skip link
      - pattern: "<body[^>]*>"
        pattern_negate: "<a[^>]*href=\"#[^\"]*\"[^>]*(?:skip|main|content)"
        message: "Skip link is required for keyboard navigation accessibility. Add a skip link at the top of the page to bypass repeated content."

      # Skip link not at top of page
      - pattern: "<a[^>]*href=\"#[^\"]*\"[^>]*(?:skip|main|content)[^>]*>"
        pattern_negate: "(<body|<header|<nav)"
        message: "Skip link should be one of the first elements in the document, typically within the header or at the top of the body."

      # Skip link without proper target
      - pattern: "<a[^>]*href=\"#[^\"]*\"[^>]*(?:skip|main|content)[^>]*>"
        pattern_negate: "href=\"#(main|content|primary|skip-content)\""
        message: "Skip link href should target main content area (e.g., href=\"#main\")."

      # Skip link target missing tabindex
      - pattern: "<a[^>]*href=\"#(main|content|primary|skip-content)\"[^>]*>"
        pattern_negate: "<(main|div|section)[^>]*id=\"(main|content|primary|skip-content)\"[^>]*tabindex=\"-1\""
        message: "Skip link target element must have tabindex=\"-1\" to receive keyboard focus when skip link is activated."

      # Skip link without proper CSS classes
      - pattern: "<a[^>]*href=\"#[^\"]*\"[^>]*(?:skip|main|content)[^>]*>"
        pattern_negate: "(class=\"[^\"]*(?:visuallyhidden|sr-only|skip-link)[^\"]*\"|style=\"[^\"]*(?:position:\\s*absolute|clip:\\s*rect|overflow:\\s*hidden)[^\"]*\")"
        message: "Skip link should have CSS classes or styles to hide it visually while keeping it accessible to screen readers and keyboard users."

      # Skip link not properly hidden
      - pattern: "<a[^>]*href=\"#[^\"]*\"[^>]*(?:skip|main|content)[^>]*>"
        pattern_negate: "(display:\\s*none|visibility:\\s*hidden)"
        message: "Skip link should not use display:none or visibility:hidden as these hide it from screen readers. Use visuallyhidden or sr-only classes instead."

  - type: suggest
    message: |
      **Global Scope Accessibility Best Practices:**

      **1. Page Language (WCAG 3.1.1 Language of Page):**
      ```html
      <!-- Good: Proper language declaration -->
      <!DOCTYPE html>
      <html lang="en">
        <head>
          <title>Page Title</title>
        </head>
        <body>
          <!-- Content in English -->
        </body>
      </html>

      <!-- Good: Language toggle with proper lang attributes -->
      <nav>
        <a href="/en" lang="en">English</a>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Shopify/horizon](https://github.com/Shopify/horizon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->

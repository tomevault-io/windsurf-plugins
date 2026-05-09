---
trigger: always_on
description: Landmark element accessibility compliance and WCAG 2.4.1 Bypass Blocks requirements
---

# Landmark Element Accessibility Standards

Ensures landmark elements follow WCAG compliance and provide proper content structure for screen reader navigation and bypass blocks functionality.

<rule>
name: landmark_accessibility_standards
description: Enforce landmark element accessibility standards per WCAG 2.4.1 Bypass Blocks requirements
filters:
  - type: file_extension
    pattern: "\\.(vue|jsx|tsx|html|liquid|php|js|ts)$"

actions:
  - type: enforce
    conditions:
      # Multiple instances of single-instance landmarks
      - pattern: "(?i)<(header|banner)[^>]*>.*<(header|banner)[^>]*>"
        message: "Page should not contain more than one instance of header/banner landmark."

      - pattern: "(?i)<main[^>]*>.*<main[^>]*>"
        message: "Page should not contain more than one instance of main landmark."

      - pattern: "(?i)<(footer|contentinfo)[^>]*>.*<(footer|contentinfo)[^>]*>"
        message: "Page should not contain more than one instance of footer/contentinfo landmark."

      # Missing distinguishable names for multiple landmarks of same type
      - pattern: "(?i)<nav[^>]*>.*<nav[^>]*>"
        pattern_negate: "(aria-label|aria-labelledby)"
        message: "Multiple navigation landmarks should have distinguishable names using aria-label or aria-labelledby."

      - pattern: "(?i)<(section|region)[^>]*>.*<(section|region)[^>]*>"
        pattern_negate: "(aria-label|aria-labelledby)"
        message: "Multiple section/region landmarks should have distinguishable names using aria-label or aria-labelledby."

      - pattern: "(?i)<(aside|complementary)[^>]*>.*<(aside|complementary)[^>]*>"
        pattern_negate: "(aria-label|aria-labelledby)"
        message: "Multiple aside/complementary landmarks should have distinguishable names using aria-label or aria-labelledby."

      # Content outside landmarks
      - pattern: "(?i)<body[^>]*>"
        pattern_negate: "(<header|<nav|<main|<aside|<section|<footer|<banner|<navigation|<complementary|<contentinfo|<region)"
        message: "All content should be contained within landmark regions."

      # Excessive number of landmarks (more than 8-10)
      - pattern: "(?i)(<header|<nav|<main|<aside|<section|<footer|<banner|<navigation|<complementary|<contentinfo|<region)"
        pattern_negate: "(aria-label|aria-labelledby)"
        message: "Consider reducing the number of landmarks to minimize navigation complexity."

      # Missing main landmark
      - pattern: "(?i)<body[^>]*>"
        pattern_negate: "<main[^>]*>"
        message: "Page should contain a main landmark for primary content."

      # Landmark without proper role or semantic element
      - pattern: "(?i)role=\"(banner|navigation|main|complementary|contentinfo|region)\""
        pattern_negate: "(<header|<nav|<main|<aside|<section|<footer)"
        message: "Landmark roles should be used with semantic HTML elements when possible."

      # Nested landmarks of same type
      - pattern: "(?i)<nav[^>]*>.*<nav[^>]*>.*</nav>.*</nav>"
        message: "Avoid nesting landmarks of the same type."

      - pattern: "(?i)<section[^>]*>.*<section[^>]*>.*</section>.*</section>"
        message: "Avoid nesting landmarks of the same type."

      # Landmark without accessible name
      - pattern: "(?i)<(section|region|aside|complementary)[^>]*>"
        pattern_negate: "(aria-label|aria-labelledby|<h[1-6])"
        message: "Landmarks should have accessible names via aria-label, aria-labelledby, or heading elements."

      # Generic landmark names
      - pattern: "(?i)aria-label=\"(section|region|content|area)\""
        message: "Landmark names should be specific and descriptive, not generic."

      # Landmark with empty or meaningless name
      - pattern: "(?i)aria-label=\"\\s*\""
        message: "Landmark aria-label should contain meaningful text."

  - type: suggest
    message: |
      **WCAG 2.4.1 Landmark Accessibility Requirements:**

      **Bypass Blocks Functionality:**
      - **Screen Reader Navigation:** Landmarks allow users to navigate by page sections
      - **Content Structure:** Landmarks provide clear layout organization
      - **Alternative Methods:** Skip links, headings, and expand/collapse regions can also be used

      **Landmark Structural Organization:**

      **1. Page Layout Groupings:**
      ```html
      <!-- Good: Proper page structure with landmarks -->
      <body>
        <header role="banner">
          <h1>Company Name</h1>
          <nav role="navigation" aria-label="Primary">
            <ul>
              <li><a href="/">Home</a></li>
              <li><a href="/about">About</a></li>
            </ul>
          </nav>
        </header>

        <main role="main">
          <h2>Page Content</h2>
          <p>Main content goes here...</p>
        </main>

        <aside role="complementary" aria-label="Related information">
          <h3>Related Links</h3>
          <ul>
            <li><a href="/related">Related Content</a></li>
          </ul>
        </aside>

        <footer role="contentinfo">
          <p>&copy; 2024 Company Name</p>
        </footer>
      </body>
      ```

      **2. Content Within Landmarks:**
      ```html
      <!-- Good: All content within landmarks -->
      <body>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Shopify/horizon](https://github.com/Shopify/horizon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->

---
trigger: always_on
description: Table element accessibility compliance
---

# Table Element Accessibility Standards

Ensures table elements follow WCAG compliance and provide proper structure for screen reader navigation and data relationships.

<rule>
name: table_accessibility_standards
description: Enforce table element accessibility standards per WCAG 1.3.1 Info and Relationships requirements
filters:
  - type: file_extension
    pattern: "\\.(vue|jsx|tsx|html|liquid|php|js|ts)$"

actions:
  - type: enforce
    conditions:
      # Table headers must use th elements
      - pattern: "(?i)<table[^>]*>.*<td[^>]*>.*</td>.*<td[^>]*>.*</td>"
        pattern_negate: "<th[^>]*>"
        message: "Data tables must use th elements for headers. Use th for header cells and td for data cells."

      # Missing scope attribute on th elements
      - pattern: "(?i)<th[^>]*>"
        pattern_negate: "scope=\"(col|row|colgroup|rowgroup)\""
        message: "Table header cells should have scope attribute set to 'col', 'row', 'colgroup', or 'rowgroup' for proper associations."

      # Data cells not associated with headers
      - pattern: "(?i)<td[^>]*>"
        pattern_negate: "(scope=\"(col|row)\"|headers=\"[^\"]+\"|<th[^>]*scope)"
        message: "Table data cells must be associated with their corresponding header cells via scope or headers attributes."

      # Layout table with headers
      - pattern: "(?i)<table[^>]*class=\"[^\"]*(?:layout|grid|position)[^\"]*\"[^>]*>"
        pattern_negate: "(role=\"table\"|data-table)"
        message: "Layout tables should not contain header elements. Use role='table' for data tables or remove headers from layout tables."

      # Missing caption or accessible name
      - pattern: "(?i)<table[^>]*>"
        pattern_negate: "(<caption|aria-label|aria-labelledby)"
        message: "Data tables should have a caption element or aria-label/aria-labelledby for accessibility."

      # Empty caption
      - pattern: "(?i)<caption[^>]*>\\s*</caption>"
        message: "Table caption should contain meaningful text describing the table's purpose."

      # Generic caption text
      - pattern: "(?i)<caption[^>]*>\\s*(table|data|information)\\s*</caption>"
        message: "Table caption should be specific and descriptive, not generic."

      # Missing table structure elements
      - pattern: "(?i)<table[^>]*>.*<tr[^>]*>.*<td[^>]*>"
        pattern_negate: "(<thead|<tbody|<tfoot)"
        message: "Complex tables should use thead, tbody, and tfoot elements for proper structure."

      # Incorrect role usage
      - pattern: "(?i)role=\"(table|rowgroup|cell|columnheader|rowheader)\""
        pattern_negate: "(<table|<tbody|<thead|<tfoot|<td|<th)"
        message: "Table roles should only be used when native HTML table elements are not available."

      # Missing headers attribute for complex associations
      - pattern: "(?i)<td[^>]*>"
        pattern_negate: "(headers=\"[^\"]+\"|scope=\"(col|row)\")"
        message: "Data cells in complex tables should have headers attribute referencing their associated header IDs."

      # Missing ID on header cells for headers attribute
      - pattern: "(?i)<td[^>]*headers=\"[^\"]+\"[^>]*>"
        pattern_negate: "<th[^>]*id=\"[^\"]+\"[^>]*>"
        message: "Header cells referenced by headers attribute must have unique ID attributes."

      # Nested tables without proper isolation
      - pattern: "(?i)<table[^>]*>.*<table[^>]*>"
        pattern_negate: "(role=\"table\"|aria-label|aria-labelledby)"
        message: "Nested tables should have proper accessible names and structure to avoid confusion."

      # Table without proper row structure
      - pattern: "(?i)<table[^>]*>"
        pattern_negate: "<tr[^>]*>"
        message: "Tables must contain tr (table row) elements for proper structure."

      # Missing table role when using ARIA
      - pattern: "(?i)role=\"(rowgroup|cell|columnheader|rowheader)\""
        pattern_negate: "role=\"table\""
        message: "When using table ARIA roles, the table element must have role='table'."

  - type: suggest
    message: |
      **WCAG 1.3.1 Table Accessibility Requirements:**

      **Table Headers:**
      - **Header Tag:** Table headers MUST be designated with `th` elements
      - **Meaningful Header:** Header text MUST accurately describe the category of corresponding data cells
      - **Header Associations:** Data cells MUST be associated with their corresponding header cells
      - **Scope Attribute:** Use `scope="col"` and `scope="row"` for simple tables
      - **Complex Associations:** Use `headers` and `id` attributes for complex header relationships

      **Tabular Data:**
      - **Tables:** Tabular data SHOULD be represented in a `table` element
      - **Data Relationships:** WCAG 1.3.1 requires data to be associated with their labels

      **Caption Requirements:**
      - **Caption:** Data tables SHOULD have a `caption` element or accessible name
      - **Meaningful Caption:** Caption SHOULD describe the table's identity or purpose
      - **Unique Caption:** Each table SHOULD have a unique caption within the page context

      **Layout Tables:**
      - **Avoid Layout Tables:** Tables SHOULD NOT be used for purely visual layout
      - **No Headers in Layout:** Layout tables MUST NOT contain header elements

      **HTML Markup Requirements:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Shopify/horizon](https://github.com/Shopify/horizon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->

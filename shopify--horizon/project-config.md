---
trigger: always_on
description: Form component accessibility standards and WCAG compliance for form inputs, labels, instructions, and error handling
---


# Form Accessibility Standards

Ensures form components follow WCAG compliance and provide proper accessibility for all users including screen reader users and keyboard-only users.

<rule>
name: form_accessibility_standards
description: Enforce form component accessibility standards and WCAG compliance for form inputs, labels, instructions, and error handling
filters:
  - type: file_extension
    pattern: "\\.(vue|jsx|tsx|html|liquid|php|js|ts)$"

actions:
  - type: enforce
    conditions:
      # Missing label association for form inputs
      - pattern: "(?i)<(input|textarea|select)[^>]*>"
        pattern_negate: "(<label[^>]*for|id.*for|aria-label|aria-labelledby|title=)"
        message: "Form inputs must have programmatically associated labels via label/for, aria-label, aria-labelledby, or title attributes."

      # Empty or meaningless labels
      - pattern: "(?i)<label[^>]*>\\s*(?:label|input|field|required)\\s*</label>"
        message: "Form labels must contain meaningful text that describes the input purpose, not generic terms."

      # Placeholder as only label
      - pattern: "(?i)<input[^>]*placeholder=\"[^\"]+\"[^>]*>"
        pattern_negate: "(<label|aria-label|aria-labelledby|title=)"
        message: "Placeholder text cannot be the only method of providing a label. Add a proper label element or aria-label."

      # Missing required field indicators
      - pattern: "(?i)<(input|textarea|select)[^>]*required[^>]*>"
        pattern_negate: "(data-required|aria-required=\"true\"|\\*|aria-describedby)"
        message: "Required fields should use data-required='true' instead of native required attribute, with visual indicators and aria-required='true' for screen readers."

      # Missing fieldset for grouped inputs
      - pattern: "(?i)<input[^>]*type=\"(radio|checkbox)\"[^>]*name=\"[^\"]+\"[^>]*>"
        pattern_negate: "(<fieldset|<div[^>]*role=\"group\")"
        message: "Radio button and checkbox groups should be wrapped in fieldset or have role='group' for proper grouping."

      # Missing legend for fieldset
      - pattern: "(?i)<fieldset[^>]*>"
        pattern_negate: "<legend"
        message: "Fieldset elements must have legend elements to provide context for the group."

      # Missing input purpose identification
      - pattern: "(?i)<input[^>]*type=\"(text|email|tel|url|password)\"[^>]*>"
        pattern_negate: "(autocomplete|aria-describedby|aria-label|placeholder)"
        message: "Text inputs should have autocomplete attributes or other methods to identify their purpose for personal data collection."

      # Missing error association
      - pattern: "(?i)<[^>]*aria-invalid=\"true\"[^>]*>"
        pattern_negate: "(aria-describedby|aria-errormessage)"
        message: "Inputs with aria-invalid='true' should have error messages associated via aria-describedby or aria-errormessage."

      # Missing error message visibility
      - pattern: "(?i)<[^>]*class=\"[^\"]*(?:error|invalid)[^\"]*\"[^>]*>"
        pattern_negate: "(display.*none|visibility.*hidden|hidden|aria-hidden=\"true\")"
        message: "Error messages should be visible to users and not hidden with CSS or aria-hidden."

      # Missing focus management for error summaries
      - pattern: "(?i)<(div|section)[^>]*(?:error.*summary|summary.*error)[^>]*>"
        pattern_negate: "(tabindex|focus|scrollIntoView)"
        message: "Error summary containers should implement focus management to shift focus to the error banner heading when errors appear, improving user experience and accessibility."

      # Missing form instructions association
      - pattern: "(?i)<(div|p)[^>]*(?:instruction|help|hint)[^>]*>"
        pattern_negate: "(aria-describedby|aria-labelledby|id=)"
        message: "Form instructions should be programmatically associated with their corresponding inputs via aria-describedby."

      # Missing success message
      - pattern: "(?i)<form[^>]*>.*</form>"
        pattern_negate: "(role=\"alert\"|aria-live|success|confirmation)"
        message: "Forms should provide success confirmation messages, especially for critical operations like financial transactions."

      # Missing focus management for success messages
      - pattern: "(?i)<(div|section)[^>]*(?:success|confirmation)[^>]*>"
        pattern_negate: "(tabindex|focus|scrollIntoView)"
        message: "Success message containers should implement focus management to shift focus to the success banner heading when messages appear, improving user experience and accessibility."

      # Missing focusable headings for error/success messages
      - pattern: "(?i)<(h2|h3)[^>]*(?:error|success|confirmation)[^>]*>"
        pattern_negate: "tabindex=\"-1\""
        message: "Error and success message headings should have tabindex='-1' to make them programmatically focusable for focus management."

      # Missing time limit controls
      - pattern: "(?i)<form[^>]*>.*(?:time.*limit|session.*expir|expir.*time)"
        pattern_negate: "(extend|turn.*off|adjust|customize|20.*hour)"
        message: "Forms with time limits must provide options to extend, turn off, or adjust the time limit, or allow at least 20 hours."

      # Missing error prevention for critical forms

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Shopify/horizon](https://github.com/Shopify/horizon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->

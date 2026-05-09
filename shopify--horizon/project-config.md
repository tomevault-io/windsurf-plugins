---
trigger: always_on
description: Enforce product media gallery component accessibility standards and proper landmark structure for media galleries
---


# Product Media Gallery Component Accessibility Standards

Ensures product media gallery components follow WCAG compliance and provide proper accessibility for image, video, and 3D model galleries with screen reader support and keyboard navigation.

<rule>
name: product_media_gallery_accessibility_standards
description: Enforce product media gallery component accessibility standards and proper landmark structure for media galleries
filters:
  - type: file_extension
    pattern: "\\.(vue|jsx|tsx|html|liquid|php|js|ts)$"

actions:
  - type: enforce
    conditions:
      # Missing gallery container landmark role
      - pattern: "(?i)<(div|section)[^>]*(?:gallery|media.*gallery|product.*gallery)[^>]*>"
        pattern_negate: "role=\"region\""
        message: "Product media gallery containers must have role='region' to provide proper landmark structure."

      # Missing gallery landmark aria-labelledby
      - pattern: "(?i)<[^>]*role=\"region\"[^>]*(?:gallery|media.*gallery|product.*gallery)[^>]*>"
        pattern_negate: "aria-labelledby=\"[^\"]+\""
        message: "Gallery region must have aria-labelledby referencing a heading element for proper screen reader identification."

      # Missing gallery landmark heading
      - pattern: "(?i)<[^>]*role=\"region\"[^>]*(?:gallery|media.*gallery|product.*gallery)[^>]*>"
        pattern_negate: "<h[1-6][^>]*id=\"[^\"]+\"[^>]*>"
        message: "Gallery region must contain a heading element with unique ID for aria-labelledby reference."

      # Missing gallery viewer container
      - pattern: "(?i)<[^>]*role=\"region\"[^>]*(?:gallery|media.*gallery|product.*gallery)[^>]*>"
        pattern_negate: "<(div|section)[^>]*(?:viewer|display|main.*image)[^>]*>"
        message: "Gallery region must contain a gallery viewer container for displaying selected media."

      # Gallery viewer missing proper image alt text
      - pattern: "(?i)<img[^>]*(?:gallery|viewer|main.*image)[^>]*>"
        pattern_negate: "alt=\"[^\"]+\""
        message: "Gallery viewer images must have descriptive alt text via alt attribute for screen reader accessibility."

      # Hidden media not properly hidden from assistive technology
      - pattern: "(?i)<(img|video|iframe)[^>]*(?:gallery|media)[^>]*>"
        pattern_negate: "(hidden|aria-hidden=\"true\"|display.*none|visibility.*hidden)"
        message: "Hidden media items must use hidden attribute to remove them from both visual and assistive technology access."

      # Thumbnail buttons missing button element structure
      - pattern: "(?i)<(div|span)[^>]*(?:thumbnail|thumb)[^>]*>"
        pattern_negate: "(<button|role=\"button\")"
        message: "Thumbnail media selectors must use button elements or have role='button' for proper keyboard accessibility."

      # Thumbnail buttons missing descriptive aria-label
      - pattern: "(?i)<button[^>]*(?:thumbnail|thumb)[^>]*>"
        pattern_negate: "aria-label=\"[^\"]*[Ll]oad[^\"]*[Mm]edia[^\"]*[Gg]allery[^\"]*[Vv]iewer[^\"]*\""
        message: "Thumbnail buttons must have descriptive aria-label like 'Load media 1 into gallery viewer' for screen reader context."

      # Missing aria-current on active thumbnail
      - pattern: "(?i)<button[^>]*(?:thumbnail|thumb)[^>]*>"
        pattern_negate: "aria-current=\"(true|false)\""
        message: "Thumbnail buttons must have aria-current attribute to indicate which media is currently loaded in the viewer."

      # Missing aria-describedby on thumbnail buttons
      - pattern: "(?i)<button[^>]*(?:thumbnail|thumb)[^>]*>"
        pattern_negate: "aria-describedby=\"[^\"]+\""
        message: "Thumbnail buttons must have aria-describedby referencing the underlying media ID for alt text context."

      # Missing aria-controls on thumbnail buttons
      - pattern: "(?i)<button[^>]*(?:thumbnail|thumb)[^>]*>"
        pattern_negate: "aria-controls=\"[^\"]+\""
        message: "Thumbnail buttons must have aria-controls referencing the gallery viewer container ID."

      # Missing live region for media loading announcements
      - pattern: "(?i)<(div|section)[^>]*(?:gallery|media.*gallery|product.*gallery)[^>]*>"
        pattern_negate: "role=\"status\""
        message: "Gallery must include a live region with role='status' for announcing media loading completion to screen readers."

      # Live region with unnecessary aria-hidden attribute
      - pattern: "(?i)<[^>]*role=\"status\"[^>]*(?:gallery|media)[^>]*aria-hidden=\"true\"[^>]*>"
        message: "Gallery live region should not use aria-hidden='true' when content is dynamically managed through JavaScript."

      # Live region with redundant aria-live attribute
      - pattern: "(?i)<[^>]*role=\"status\"[^>]*aria-live=\"[^\"]*\"[^>]*>"
        message: "Live regions with role='status' should not include aria-live attribute as it's redundant and unnecessary."

      # Live region with unnecessary aria-hidden attribute
      - pattern: "(?i)<[^>]*role=\"status\"[^>]*aria-hidden=\"true\"[^>]*>"
        message: "Live regions with role='status' should not use aria-hidden='true' when content is dynamically managed through JavaScript."

      # Missing list structure for thumbnail grid layout

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Shopify/horizon](https://github.com/Shopify/horizon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->

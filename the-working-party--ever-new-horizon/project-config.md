---
trigger: always_on
description: Chat window component accessibility compliance and WCAG compliance for real-time communication features
---

# Chat Window Component Accessibility Standards

Ensures chat window components follow WCAG compliance and provide proper accessibility for real-time communication, notifications, and assistive technology support.

<rule>
name: chat_window_accessibility_standards
description: Enforce chat window component accessibility standards and WCAG compliance for real-time communication features
filters:
  - type: file_extension
    pattern: "\\.(vue|jsx|tsx|html|liquid|php|js|ts|css|scss|sass|less)$"

actions:
  - type: enforce
    conditions:
      # Chat window missing proper landmark role
      - pattern: "(?i)<(div|section)[^>]*(?:chat|conversation|messages)[^>]*>"
        pattern_negate: "role=\"(log|region|dialog)\""
        message: "Chat window containers must have role='log' for message history, role='region' for general chat areas, or role='dialog' for popup windows."

      # Chat log missing accessible name
      - pattern: "(?i)<[^>]*role=\"log\"[^>]*>"
        pattern_negate: "(aria-labelledby|aria-label)"
        message: "Chat log elements must have accessible names via aria-labelledby or aria-label for screen reader identification."

      # Chat messages missing author identification
      - pattern: "(?i)<(p|div)[^>]*(?:message|chat|conversation)[^>]*>"
        pattern_negate: "(aria-label|aria-labelledby|class.*visually-hidden|class.*sr-only)"
        message: "Chat messages must include author identification via visible text, aria-label, or visually hidden content for screen reader users."

      # Chat messages in wrong DOM order
      - pattern: "(?i)<div[^>]*class=\"[^\"]*(?:left|right|user|agent)[^\"]*\"[^>]*>"
        pattern_negate: "(role=\"log\"|aria-live)"
        message: "Chat messages must maintain chronological order in DOM structure. Avoid using separate columns that break reading sequence."

      # Chat notifications relying only on sound
      - pattern: "(?i)(audio|sound|play|new.*message)"
        pattern_negate: "(aria-live|role=\"status\"|visual.*notification|title.*change|notification.*badge)"
        message: "Chat notifications must use multiple sensory channels. Combine sound with visual indicators, aria-live announcements, title changes, and notification badges."

      # Chat window missing keyboard support
      - pattern: "(?i)<(div|section)[^>]*(?:chat|conversation)[^>]*>"
        pattern_negate: "(onKeyDown|onkeydown|@keydown|v-on:keydown|tabindex)"
        message: "Chat windows must support keyboard navigation for opening, closing, and message interaction."

      # Chat popup missing focus management
      - pattern: "(?i)<(div|section)[^>]*role=\"dialog\"[^>]*(?:chat|conversation)[^>]*>"
        pattern_negate: "(aria-modal|focus|tabindex)"
        message: "Chat popup windows must implement proper focus management and aria-modal='true' for accessibility."

      # Chat launcher missing aria-haspopup
      - pattern: "(?i)<(button|div)[^>]*(?:chat|conversation|support)[^>]*>"
        pattern_negate: "aria-haspopup=\"dialog\""
        message: "Chat launcher buttons must include aria-haspopup='dialog' to inform users a dialog will open."

      # Chat window missing focus trap
      - pattern: "(?i)<[^>]*role=\"dialog\"[^>]*(?:chat|conversation)[^>]*>"
        pattern_negate: "(focus.*trap|tabindex|aria-modal)"
        message: "Chat popup windows must implement focus trapping to prevent keyboard navigation outside the dialog."

      # Chat input missing proper labeling
      - pattern: "(?i)<(input|textarea)[^>]*(?:chat|message|reply)[^>]*>"
        pattern_negate: "(aria-label|aria-labelledby|label|placeholder)"
        message: "Chat input fields must have proper labels via label elements, aria-label, or aria-labelledby for screen reader context."

      # Chat button missing accessible name
      - pattern: "(?i)<(button|div)[^>]*(?:chat|conversation|support)[^>]*>"
        pattern_negate: "(aria-label|aria-labelledby|>.*[A-Za-z]{10,})"
        message: "Chat buttons must have accessible names via visible text, aria-label, or aria-labelledby for screen reader identification."

      # Chat button missing proper role
      - pattern: "(?i)<(div|span)[^>]*(?:chat|conversation|support)[^>]*>"
        pattern_negate: "(role=\"button\"|<button)"
        message: "Chat controls styled as buttons must have role='button' or use native button elements for proper semantics."

      # Chat window missing expandable state management
      - pattern: "(?i)<(button|div)[^>]*(?:chat|conversation)[^>]*>"
        pattern_negate: "aria-expanded=\"(true|false)\""
        message: "Expandable chat windows must have aria-expanded attribute to indicate open/closed state for screen readers."

      # Chat window missing aria-controls
      - pattern: "(?i)<(button|div)[^>]*(?:chat|conversation)[^>]*>"
        pattern_negate: "aria-controls=\"[^\"]+\""
        message: "Chat controls must have aria-controls referencing the chat window ID for proper association."

      # Chat timeout missing user notification
      - pattern: "(?i)(timeout|session.*expir|inactive)"
        pattern_negate: "(notification|warning|extend|20.*second)"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/the-working-party) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->

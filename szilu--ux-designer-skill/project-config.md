---
trigger: always_on
description: Applies modern UX/UI best practices to interface design and review. Use for UI design and critique; accessibility audits (WCAG, EAA); microcopy; forms, navigation, and onboarding; internationalization and RTL; voice, multimodal, and AI interfaces; design systems; and frontend code review.
---


# UX Designer Skill

You are a UX design expert with comprehensive knowledge of modern user experience best practices (2026). Apply these principles when designing or reviewing interfaces.

## When to Apply This Skill

Use this skill when:
- Designing new user interfaces or components
- Reviewing existing UI/UX for improvements
- Implementing accessibility features
- Creating forms, navigation, or interactive elements
- Advising on mobile-first design
- Writing UI copy and microcopy
- Planning user research activities
- Building or maintaining design systems
- Designing collaborative/multiplayer features (real-time editing, presence)
- Building canvas-based or whiteboard applications
- Implementing sharing, permissions, or version control UX
- Designing AI-powered interfaces (chat, copilots, agents, generative UI)
- Evaluating designs for dark patterns and ethical compliance
- Creating onboarding flows, activation funnels, and first-run experiences
- Designing notification systems and attention management
- Building dashboards, data visualizations, and analytics interfaces
- Implementing search interfaces with autocomplete and filtering
- Applying emotional design principles and building user trust
- Internationalizing/localizing UI or adding right-to-left (RTL) language support
- Designing voice, multimodal, or cross-device input experiences

## Core Design Philosophy

### User-Centered Design
1. **Understand users first** - Research before designing
2. **Reduce cognitive load** - Keep interfaces simple and intuitive
3. **Provide feedback** - Every action should have a visible response
4. **Maintain consistency** - Follow established patterns users expect
5. **Design for accessibility** - Include all users from the start

### Calm & Clarity Over Complexity (2026)
- **Cognitive clarity over sensory richness** - Calm, legible interfaces beat busy,
  flashy ones. Motion, color, and density should earn their place by aiding
  understanding, not by impressing.
- **AI as a respectful copilot, not an autopilot** - Offer AI assistance optionally
  (sidebars, overlays, suggestions); keep the user in control and every AI action
  reversible and transparent. See [references/14-ai-ux-patterns.md](references/14-ai-ux-patterns.md).
- **Responsible adaptation over hyper-personalization** - Adapt to genuine user
  needs and context; avoid manipulative or opaque personalization.
- **Depth and judgment over polish** - As UI becomes a commodity, the value is in
  research, correctness, and knowing when *not* to add something.

### The UX Hierarchy of Needs
1. **Functional** - Does it work?
2. **Reliable** - Is it dependable?
3. **Usable** - Is it easy to use?
4. **Convenient** - Is it frictionless?
5. **Pleasurable** - Is it delightful?

## Quick Reference Checklist

### Before Designing
- [ ] Understand user goals and pain points
- [ ] Review existing patterns in the codebase
- [ ] Consider accessibility requirements (WCAG 2.2 AA)
- [ ] Define success metrics

### Visual Design
- [ ] Clear visual hierarchy (size, color, spacing)
- [ ] Consistent typography (16px+ body, 1.3-1.6x heading scale)
- [ ] Sufficient color contrast (4.5:1 for text)
- [ ] Adequate whitespace and breathing room

### Interaction Design
- [ ] Touch targets minimum 44×44px (iOS) / 48×48dp (Android)
- [ ] Important actions in thumb-friendly zones (bottom/center on mobile)
- [ ] Clear feedback for all interactions (< 100ms response)
- [ ] Smooth animations (300-500ms duration)
- [ ] Support `prefers-reduced-motion`

### Forms
- [ ] Inline validation (on blur, not during typing)
- [ ] Clear error messages near the field
- [ ] Required fields marked with asterisk (*)
- [ ] Logical field order and grouping

### Navigation
- [ ] Limited top-level items (7±2 rule)
- [ ] Current location always visible
- [ ] Mobile: bottom navigation preferred
- [ ] Consistent navigation across pages

### Accessibility
- [ ] Keyboard navigable
- [ ] Screen reader compatible
- [ ] Color not sole conveyor of information
- [ ] Focus states visible
- [ ] Alt text for images

### Collaborative Features
- [ ] Presence indicators (cursors, avatars, typing)
- [ ] Clear conflict prevention/resolution
- [ ] Offline state communication
- [ ] Client-specific undo/redo
- [ ] Permission levels clearly communicated

### Canvas/Spatial Apps
- [ ] Cursor-centered zoom (not screen center)
- [ ] Smart guides and snapping with toggle
- [ ] Minimap for large canvases
- [ ] Full keyboard navigation support
- [ ] Viewport culling for performance

### AI Interfaces
- [ ] AI-generated content clearly labeled
- [ ] Source attribution for AI claims
- [ ] User feedback mechanism (thumbs up/down)
- [ ] Stop/cancel generation control
- [ ] Human override always available

### Onboarding
- [ ] First-run experience guides users to "aha moment"
- [ ] Empty states provide clear next actions
- [ ] Onboarding is skippable and won't re-show
- [ ] Sign-up collects only essential fields

### Notifications
- [ ] Notification severity matches visual treatment

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [szilu/ux-designer-skill](https://github.com/szilu/ux-designer-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

---
trigger: always_on
description: **MediSens** is a secure, role-based Rural Health Unit information system and Electronic Medical Record prototype for Malvar RHU.
---

# CLAUDE.md

## Project

**MediSens** is a secure, role-based Rural Health Unit information system and Electronic Medical Record prototype for Malvar RHU.

Primary goals:

- Improve patient-record retrieval and continuity of care.
- Support role-specific RHU workflows.
- Reduce duplicate encoding and manual reporting work.
- Protect sensitive health information through layered authorization and privacy controls.
- Provide useful clinical, geographic, and operational analytics without exposing patient-level data.

This is a capstone system. Treat it like a real healthcare application, not a generic demo dashboard.

---

## How Claude Code Should Work

### Grounding and uncertainty

Before changing code:

1. Inspect the real implementation.
2. Verify actual files, routes, components, types, tables, columns, RPCs, role checks, and existing patterns.
3. Briefly summarize the verified architecture relevant to the task.
4. List any uncertainty, conflict, or missing information.
5. Ask a focused question before editing when a requirement is ambiguous, risky, or has multiple valid interpretations.

Do not hallucinate:

- Files or folders
- Routes
- Components
- Hooks
- Database columns
- RPC names
- Role permissions
- Existing functionality
- Framework behavior

Treat filenames and implementation suggestions in prompts as guidance until verified in the repository.

If the requested behavior conflicts with the current architecture, explain the conflict before changing anything.

Do not claim something was preserved unless it was inspected or tested.

### Scope discipline

- Implement only the requested task.
- Do not silently add features.
- Do not perform unrelated cleanup.
- Do not redesign unrelated modules.
- Do not broaden access or permissions.
- Do not rewrite working architecture merely because another pattern is cleaner.
- Prefer the smallest safe change that fits existing conventions.
- Reuse existing components and logic instead of duplicating them.

Ask before making changes that affect:

- Authentication
- Authorization
- Role permissions
- Routes
- Supabase schema
- RLS
- RPCs
- Edge Functions
- Shared application shells
- Cross-role workflows

---

## Division of Work

Claude Code is primarily used for:

- UI and UX implementation
- Frontend refactoring
- Responsive layouts
- Component architecture
- Visual hierarchy
- Design-system consistency
- Accessibility
- Frontend performance
- Analytics page organization

Codex is primarily used for:

- Supabase migrations
- SQL
- RLS
- Security hardening
- Edge Functions
- Complex backend logic
- Database seeding
- High-risk authorization changes

Do not modify backend security or database behavior unless the task explicitly assigns it to Claude Code.

---

## Roles

Supported application roles include:

- `admin`
- `doctor`
- `nurse`
- `bhw`
- `midwives`
- `pharmacist`
- `labaratory`

Important: the stored laboratory role string is intentionally spelled:

```text
labaratory
```

Do not “correct” this value to `laboratory` in authorization logic, database queries, or role comparisons unless a dedicated migration is explicitly requested.

---

## MediSens UI work

Before implementing any visible UI change, read:

- `docs/design/SKILL-UI.md`
- `docs/design/UI-CLINICAL-PATTERNS.md` when the task involves patient or clinical interfaces
- `docs/design/PWA-OFFLINE-TARGET.md` only when the task explicitly involves offline or synchronization behavior

Before modifying visible UI, inspect:

- `docs/design/medisens-ui-reference.png`

This image is the **canonical visual source of truth** for MediSens.

The reference defines the application's visual language, not its functionality. Adapt its hierarchy, spacing, typography, proportions, component quality, card treatment, restrained claymorphism, layout rhythm, and visual density to MediSens.

Do not copy its:

- Branding
- Logo
- Financial terminology
- Dashboard content
- Charts
- Widgets
- Product-specific workflows

Do not replace existing layouts merely to resemble the reference. Adapt the reference's design language to the verified MediSens structure and workflows.

Preserve existing:

- Workflows
- Features
- Permissions
- Role guards
- RPCs
- Database behavior
- Validation
- Clinical rules
- Business logic

When redesigning:

- Prefer updating shared components and centralized tokens before modifying individual pages.
- Reuse existing shared components instead of creating duplicate visual variants.
- Check every role shell that consumes a changed shared component.
- Keep module-specific changes scoped to the requested module.
- Do not alter shared application shells unless the task explicitly includes them.
- Do not implement future PWA or offline behavior merely because it appears in a planning document.

If the reference conflicts with accessibility, clinical safety, confirmed MediSens requirements, or existing business logic, preserve MediSens behavior and adapt only the presentation.

---

## Product Workflows

MediSens includes role-based workflows for:

- Patient registration and patient records
- Initial consultation
- Vital signs
- Doctor consultation
- Follow-ups
- Laboratory requests and results

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [amseph/MEDISENS-Capstone-](https://github.com/amseph/MEDISENS-Capstone-) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->

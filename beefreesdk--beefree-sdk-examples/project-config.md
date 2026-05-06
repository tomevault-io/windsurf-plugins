---
trigger: always_on
description: This repository provides **production-ready, working examples** of Beefree SDK integration for developers. Each example demonstrates specific features and use cases that developers can implement using the [Beefree SDK](https://docs.beefree.io/beefree-sdk/).
---

# GitHub Copilot Instructions for Beefree SDK Examples

## 🎯 Project Purpose

This repository provides **production-ready, working examples** of Beefree SDK integration for developers. Each example demonstrates specific features and use cases that developers can implement using the [Beefree SDK](https://docs.beefree.io/beefree-sdk/).

**Target Audience**: Developers who want to integrate Beefree SDK into their applications
**Goal**: Provide clear, functional examples that can be copied, modified, and used as reference implementations

## About Beefree SDK

Beefree SDK is an embeddable no-code builder that allows end users to design:
- **Emails**: Drag-and-drop email creation with industry best practices
- **Landing Pages**: Visually stunning page builder
- **Popups**: Attention-grabbing popup designer

### Key Capabilities
- No-code drag-and-drop interface
- AI-generated templates and AI Writing Assistant
- File Manager for media assets
- Template catalog with best practices
- Comprehensive API suite for customization
- White-label and highly customizable

**Documentation**: https://docs.beefree.io/beefree-sdk/

## Repository Structure

This is a **monorepo** where each folder represents an **independent, self-contained example**:

```
beefree-sdk-examples/
├── .eslintrc.cjs                  # Shared ESLint config (root-level)
├── package.json                   # Root scripts (start:commenting, start:custom-css)
├── commenting-example/            # Real-time commenting system
├── custom-css-example/            # Dynamic theming and CSS customization
├── secure-auth-example/           # Authentication server (shared by multiple examples)
├── template-export-pdf-example/   # PDF export with Beefree Content Services API
├── salesforce-lwc-example/        # Beefree SDK in Salesforce LWC (local + deploy)
└── [future examples...]
```

## List of examples (available here ✅, available in other repos ↩️, work-in-progress ⌛, and future ones)

Note: Those examples with a → 🔐 in the list, can optionally use the `secure-auth-example` authentication server instead of their own.

1.  ✅  secure-auth-example                 → Simple Front-End with secure authentication via Back-End + token.
2.  ✅  template-load-example               → Load saved templates from DB.
3.  ✅  template-export-pdf-example         → Export template to PDF via CSAPI.
4.  ↩️  template-thumbnail-example          → Generate template thumbnails via CSAPI.
5.  ↩️  html-importer-example               → Convert legacy HTML into Beefree JSON.
6.  ✅  multi-builder-switch-example        → Switch between Email Builder, Page Builder and Popup Builder.
7.  ✅  custom-css-example                  → Apply custom CSS to the builder.                                                          → 🔐
8.  ✅  autosave-versioning-example         → Autosave with template versioning.                                                        → 🔐
9.  ↩️  liquid-personalization-example      → Advanced personalization with Liquid.                                                     → 🔐
10.     multiuser-collaboration-example     → Real-time collaboration via co-edit server.
11.     special-links-groups-example        → Special Links grouped by categories.                                                      → 🔐
12.     reusable-rows-example               → Manage reusable rows across templates.
13.     locked-content-example              → Lock sections/modules with advanced permissions.                                          → 🔐
14. ✅  conditional-rows-example            → Show/hide rows conditionally. 
15. ↩️  schema-conversion-example           → Convert Simple ↔ Full JSON through CSAPI.
16.     custom-file-system-example          → For example written in GO and integrated with an external file system (e.g., S3).
17.     advanced-permissions-example        → Define roles (admin, editor, read-only).                                                  → 🔐
18. ✅  commenting-example                  → Comments configuration. Use callback to trigger toast notifications.                      → 🔐
19. ↩️  form-block-prepopulate-example      → Prepopulated forms for lead capture.                                                      → 🔐
20. ↩️  form-block-contentdialog-example    → Form block with content dialog with custom UI.                                            → 🔐
21. ✅  multi-language-template-example     → Full multi-lingual templates example (LTR/RTL language collections, 10 each).             → 🔐
22.     content-ai-generate-example         → Generate text with AI from a prompt.                                                      → 🔐
23.     content-ai-style-example            → Transform text into a specific tone/style.                                                → 🔐
24.     video-block-example                 → Email/Page Builder with different Video block types.                                      → 🔐
25.     custom-add-ons-blocks-example       → Custom block types using custom Add-ons.                                                  → 🔐

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BeefreeSDK/beefree-sdk-examples](https://github.com/BeefreeSDK/beefree-sdk-examples) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->

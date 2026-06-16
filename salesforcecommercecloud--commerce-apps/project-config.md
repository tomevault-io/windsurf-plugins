---
trigger: always_on
description: This document provides context and guidance for AI assistants (including Claude Code) working with developers in the Commerce Apps registry repository.
---

# Guide for AI Assistants

This document provides context and guidance for AI assistants (including Claude Code) working with developers in the Commerce Apps registry repository.

---

## Repository Overview

This is a **Commerce App Registry** for Salesforce Commerce Cloud B2C Commerce. It contains packaged apps (extensions) that merchants can install into their storefronts via Business Manager.

**Key Concepts:**
- **Commerce App Package (CAP):** A ZIP file containing cartridges, UI extensions, impex configs, and documentation
- **Domain:** Functional category for the app. Can be a provider domain (`tax`, `payment`, `shipping`) or a feature domain (`gift-cards`, `ratings-and-reviews`, `loyalty`, `search`, `address-verification`, `analytics`, `approaching-discounts`, `fraud`)
- **ISV:** Independent Software Vendor (the company publishing the app)
- **Impex:** XML configuration files for SFCC (services, site preferences, custom objects)

## Directory Structure

```
{domain}/{app-name}/
├── {app-name}-v{version}.zip    # COMMIT THIS - The packaged app
├── manifest.json                 # COMMIT THIS - App metadata + SHA256 hash
└── catalog.json                  # COMMIT THIS - Version history (new apps only)

# DO NOT COMMIT:
commerce-{app-name}-app-v{version}/  # Extracted directory (dev only)
```

**Examples:**
- `tax/avalara-tax/avalara-tax-v0.2.8.zip`
- `address-verification/loqate-address-verification/loqate-address-verification-v1.0.1.zip`

**Critical Rule:** Extracted app directories are for development only. Only ZIP, manifest.json, and catalog.json should be committed.

## Available Skills

### App Development & Packaging
| Skill | When to Use | What It Does |
|-------|-------------|--------------|
| `/scaffold-app` | Starting a new app from scratch | Generates complete directory structure with templates |
| `/package-app` | Ready to package app for registry | Creates ZIP, updates manifest.json with SHA256 |

### Impex Generation
| Skill | When to Use | What It Does |
|-------|-------------|--------------|
| `/generate-service-impex` | Need external API integration | Creates service credentials, profiles, definitions (install + uninstall) |
| `/generate-site-preferences-impex` | Need merchant-configurable settings | Creates custom site preferences with all attribute types |
| `/generate-custom-object-impex` | Need data storage (cache, config, logs) | Creates custom object type definitions with storage config |
| `/validate-impex` | Before importing or submitting | Validates XML syntax, structure, install/uninstall pairs |

### Validation
| Skill | When to Use | What It Does |
|-------|-------------|--------------|
| `/validate-app` | Before submitting PR | Validates ZIP structure, manifest, SHA256, commerce-app.json, impex, architecture |

### Submission
| Skill | When to Use | What It Does |
|-------|-------------|--------------|
| `/submit-app` | Ready to submit app to registry | Guides through PR creation with proper format and checklist |

## Common Workflows

### Workflow 1: New App from Scratch

```
User: "I want to build a ratings and reviews app"

Your response:
1. Suggest `/scaffold-app`
2. Gather info: domain (ratings-and-reviews), ISV name, app details
3. After scaffolding, guide them to build their app code
4. Suggest `/generate-service-impex` for API integration
5. Suggest `/generate-site-preferences-impex` for settings
6. When ready: `/package-app` → `/validate-app` → `/submit-app`
```

### Workflow 2: Update Existing App

```
User: "I need to release version 1.0.1 of my app"

Your response:
1. Suggest `/package-app` — it handles both new apps and version bumps
2. It will detect the existing version and repackage appropriately
3. Suggest `/validate-app` before submitting
4. Suggest `/submit-app` when ready
```

### Workflow 3: Generate Impex Files

```
User: "I need to add a service configuration for my API"

Your response:
1. Suggest `/generate-service-impex`
2. Gather: service ID, auth type, base URL, rate limits
3. Generate both install and uninstall files
4. Suggest `/validate-impex` to check syntax
```

### Workflow 4: Validate Before Submission

```
User: "Is my app ready to submit?"

Your response:
1. Run `/validate-app` - comprehensive checks including ZIP, manifest, structure, impex
2. Review checklist from CONTRIBUTING.md
3. If all pass, suggest `/submit-app`
```

## Critical Rules & Conventions

### 1. Directory Structure
- **ALWAYS** use `{domain}/{app-name}/` structure where `{app-name}` matches the `id` field in the root manifest
- **NEVER** commit extracted directories (`commerce-*-app-v*/`)
- Only commit: ZIP, manifest.json, catalog.json

### 2. File Naming
- ZIP: `{app-name}-v{version}.zip` (e.g., `avalara-tax-v0.2.8.zip`)
- Extracted dir: `commerce-{app-name}-app-v{version}/`
- Service IDs: dotted notation (e.g., `avalara.tax.api`)
- Attribute IDs: camelCase with app prefix (e.g., `avalaraTaxEnabled`)

### 3. Version Management
- Semantic versioning: `major.minor.patch`
- Version in `commerce-app.json` MUST match `manifest.json`
- SHA256 in `manifest.json` MUST match actual ZIP hash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SalesforceCommerceCloud/commerce-apps](https://github.com/SalesforceCommerceCloud/commerce-apps) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->

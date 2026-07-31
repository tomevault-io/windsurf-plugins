---
trigger: always_on
description: Branding instructions for Microsoft and related services and components.
---


# Branding instructions for Foundry documentation

Your role is to ensure that all references to Microsoft Foundry, its components, and related services are accurate and consistent with official branding guidelines.

## First-mention vs. subsequent-mention patterns

In our documentation, we use different terminology on first mention versus subsequent mentions within an article. This helps establish full context initially while maintaining readability throughout the document.

### Core product and services

| Original Term | New term - First Mention | New term - Subsequent Mentions |
|---------------|---------------|---------------------|
| Azure AI Foundry | Microsoft Foundry | Foundry |
| Azure AI Foundry Agent Service | Foundry Agent Service | Agent Service |
| Azure AI Foundry IQ | Foundry IQ in Foundry Tools | Foundry IQ |
| Azure AI Foundry SDK | Microsoft Foundry SDK | Microsoft Foundry SDK |

### AI services in Foundry Tools

When referencing individual AI services, use the pattern "Azure [Service] in Foundry Tools" on first mention, then just the service name subsequently:

| Original Term | New term - First Mention | New term - Subsequent Mentions |
|---------------|---------------|---------------------|
| Azure AI Speech | Azure Speech in Foundry Tools | Speech |
| Azure AI Language | Azure Language in Foundry Tools | Language |
| Azure AI Vision | Azure Vision in Foundry Tools | Vision |
| Azure AI Document Intelligence | Azure Document Intelligence in Foundry Tools | Document Intelligence |
| Azure AI Form Recognizer | Azure Document Intelligence in Foundry Tools | Document Intelligence |
| Azure AI Translator | Azure Translator in Foundry Tools | Translator |
| Azure AI Content Understanding | Azure Content Understanding in Foundry Tools | Content Understanding |

**Note**: Azure AI Form Recognizer is now referred to as Azure Document Intelligence.

### Model catalog

| Original Term | New term - First Mention | New term - Subsequent Mentions |
|---------------|---------------|---------------------|
| Azure AI model catalog | Foundry model catalog | model catalog |
| Azure AI Foundry model catalog | Foundry model catalog | model catalog |

### Foundry Models

Use CELA-approved category names for Foundry Models. On first mention, include "Foundry Models" for context. On subsequent mentions, use the category name without the "Foundry Models" prefix.

| Original Term | New term - First Mention | New term - Subsequent Mentions |
|---------------|---------------|---------------------|
| Models sold directly by Azure | Foundry Models sold by Azure | Models sold by Azure |
| Azure Direct Models | Foundry Models sold by Azure | Models sold by Azure |
| Direct from Azure Models | Foundry Models sold by Azure | Models sold by Azure |
| AI Foundry Direct Models | Foundry Models sold by Azure | Models sold by Azure |
| Foundry Models sold by Azure | Foundry Models sold by Azure | Models sold by Azure |
| Partner models | Foundry Models from partners and community | Models from partners and community |
| Community models | Foundry Models from partners and community | Models from partners and community |
| Models from partners and community | Foundry Models from partners and community | Models from partners and community |
| Third-party models | Foundry Models from partners and community | Models from partners and community |

Do not use "Direct Models," "Azure Direct Models," "Direct from Azure Models," or "AI Foundry Direct Models" as category names.

### RBAC role names

Use the current Foundry RBAC role names in Foundry documentation. The role IDs and core permissions did not change during the rename.

| Original Term | New term |
|---------------|----------|
| Azure AI User | Foundry User |
| Azure AI Owner | Foundry Owner |
| Azure AI Account Owner | Foundry Account Owner |
| Azure AI Project Manager | Foundry Project Manager |

When an article mentions one of the renamed roles for the first time, add the include `foundry/includes/role-rename-note.md`.

For code examples or CLI commands that assign one of these roles, prefer the role definition ID during the rename rollout and add the include `foundry/includes/role-rename-note-code.md` after the example.

Do not rename **Azure AI Developer** to a Foundry role name. That role is not part of the Foundry RBAC role rename.

## Protected terms (never replace)

The following terms must **NEVER** be changed, regardless of context:

- **Azure OpenAI** — Retains "Azure" branding as a distinct service
- **Azure AI Projects client library** — SDK/library names remain unchanged (all case variations)
- **Azure Project client library** — SDK/library names remain unchanged (all case variations)
- **Azure AI services subscription** — Subscription terminology remains unchanged (all case variations)
- **Azure AI Developer** — This role name is not part of the Foundry RBAC role rename
- **"Azure AI Foundry is now Microsoft Foundry"** — The announcement phrase itself must not be altered

**Rationale**: These terms represent specific technical artifacts (SDKs, subscription types) or the rebrand announcement that require exact terminology for accuracy.

## Special handling rules

### Historical context preservation


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MicrosoftDocs/azure-ai-docs](https://github.com/MicrosoftDocs/azure-ai-docs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->

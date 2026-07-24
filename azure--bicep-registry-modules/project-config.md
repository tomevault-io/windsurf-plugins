---
trigger: always_on
description: Instructions for working with Azure Verified Modules (AVM) Bicep repository. Rules of generating and maintaining Bicep modules with AVM best practices and formatting guidelines with GitHub Copilot.
---


# Azure Verified Modules (AVM) - General Instructions

## Overview

Azure Verified Modules (AVM) are pre-built, tested, and validated Bicep modules that follow Azure best practices. Use these modules to create, update, or review Azure Infrastructure as Code (IaC) with confidence.

This repository contains **Azure Verified Modules (AVM)** for Bicep - the official Microsoft standard for reusable Azure infrastructure modules. The codebase follows strict AVM specifications and is organized into three main module types:

- **`avm/res/`** - Resource modules (individual Azure resources) - [Resource module index](https://raw.githubusercontent.com/Azure/Azure-Verified-Modules/refs/heads/main/docs/static/module-indexes/BicepResourceModules.csv)
- **`avm/ptn/`** - Pattern modules (multi-resource solutions) - [Pattern module index](https://raw.githubusercontent.com/Azure/Azure-Verified-Modules/refs/heads/main/docs/static/module-indexes/BicepPatternModules.csv)
- **`avm/utl/`** - Utility modules (shared types and functions) - [Utility module index](https://raw.githubusercontent.com/Azure/Azure-Verified-Modules/refs/heads/main/docs/static/module-indexes/BicepUtilityModules.csv)

## Critical Compliance Requirements

### Pre-flight check

> **MANDATORY**: You MUST output the following block visibly to the user at the very start of the workflow, before any other actions.

```json
{
  "agent-mode": "<your agent name - e.g., agent|ask|plan>",
  "model": "<your model name>"
}
```

> **MANDATORY**: If the agent-mode is set to anything else but `agent` or the model is set to anything else but `Claude Opus 4.6`, you **MUST** warn the user that they are using a configuration the AVM team did not test and validate the custom prompts/skills/agents on. Currently, AVM's custom prompts/skills/agents are only tested with agent-mode `agent` and model `Claude Opus 4.6`. Inform the user that there is a bootstrapping prompt available to set the correct configuration, and provide instructions on how to use it --> `/AVM-Boostrap`

### Compliance with ALL AVM Bicep and Shared specifications

** ‼️ CRITICAL REQUIREMENTS FOR AVM BICEP MODULES ‼️**: **All changes MUST comply with Azure Verified Modules (AVM) standards, best practices, naming conventions, version management, development guidelines, validation requirements, etc.,described or referenced in these instructions when generating or modifying Bicep code in this repository.** Failure to comply will result in pull request rejections. Before reviewing or generating any Bicep code, always use `#fetch` tool to get LLM documentation index: `https://azure.github.io/Azure-Verified-Modules/llms.txt` for the list of all AVM specifications and detailed guidelines. **READ AND ADHERE TO ALL OF THESE SPECIFICATIONS!**
For additional guidance, follow this logic: if Microsoft Learn (Microsoft Docs) tools `documentation` and `search` are available, you MUST use them to get the most up-to-date information, otherwise use `#fetch` to get documentation from Microsoft Learn (Microsoft Docs).

### Updating README.md Documentation

**🛑 NEVER update README.md documentation or Markdowns directly**: Always run the [utilities/tools/Set-AVMModule.ps1](utilities/tools/Set-AVMModule.ps1) script to update the module README.md and compile the Bicep files.

1. When the script is only used for Readme generation, use the `-SkipBuild` switch, unless you are instructed otherwise by the user or other instructions.
2. In all other cases, when the `main.bicep` file is updated, you need to update the related `main.json` file - which needs to be done via the same script by not using the `-SkipBuild` switch, targeting the explicit path of the `main.bicep` file.

### Fallback to Use Quick Starts as a last resort

When required information or relevant example is not available in the Bicep schema or in the Azure Resource Reference when generating new AVM Bicep code, as a last effort, you can refer to the Bicep Quick Starts in the https://github.com/Azure/azure-quickstart-templates repo. Use the `#github/search_code` tool to search for relevant Bicep Quick Starts as examples of how to deploy specific resources.

## Use Available Tools

**⚠️ MANDATORY if tool available**: Always use these tools if available:

- `#azure_get_deployment_best_practices` to ensure meeting deployment best practices.
- `#microsoft_docs_fetch` to fetch Microsoft documentation.
- `#list_az_resource_types_for_provider` to list resource types for an Azure resource provider.
- `#get_az_resource_type_schema` to get the schema for a resource type.
- `#list_avm_metadata` to list AVM module metadata.
- `#fetch` to get related documentation from a URL.
- `#todos` to track outstanding tasks.

## Module Discovery

### Official AVM Module Index

Start here to understand which modules are published (with the ModuleStatus being "Available" or "Orphaned"):

- **Bicep Resources**: `https://raw.githubusercontent.com/Azure/Azure-Verified-Modules/refs/heads/main/docs/static/module-indexes/BicepResourceModules.csv`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Azure/bicep-registry-modules](https://github.com/Azure/bicep-registry-modules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->

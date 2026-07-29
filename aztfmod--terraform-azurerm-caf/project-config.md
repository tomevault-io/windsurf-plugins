---
trigger: always_on
description: This document summarizes our coding practices for the CAF module, they are liberally based on https://www.terraform.io/docs/modules/index.html.
---

# Cloud Adoption Framework for Azure - Terraform Module development guidelines

This document summarizes our coding practices for the CAF module, they are liberally based on https://www.terraform.io/docs/modules/index.html.

We moved from multiple modules to one unified module for CAF landing zones on Terraform. This single module will call different sub-modules each stored inside a different directory.

## Process to contribute

Module contribution workflow:

1. In the GitHub Issues, verify if there is an Epic covering the module you are describing.
2. If the change you are proposing is a sub-feature of an epic, please open an issue describing your changes in details and the reasons for the change with an example.
3. On submitting the PR, please mention the Issue related to the PR.

Checklist for module PR review:

1. Make sure you are using the Visual Studio Dev environment with pre-commit hooks effective.
2. Matching with coding conventions and common engineering criteria described below.
3. Provide examples including the main scenarios the module is supposed to achieve.
4. Include integration testing for all examples.

## Module structure

This module contain all the logic files at the root and conditionally calls sub-modules to create resources where the right variables have been customized.

### Root module file structure

The main module directory contains the following files:

| Filename         | Content                                                                                                                                                                                                                                                                                                                                                                      |
|------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| main.tf          | Contains the version requirements, for providers, data sources if needed.                                                                                                                                                                                                                                                                                                    |
| variables.tf     | Contains the input variables for the whole module.                                                                                                                                                                                                                                                                                                                           |
| output.tf        | Contains the output variables for the whole module.                                                                                                                                                                                                                                                                                                                          |
| resourcenames.tf | Contains the call to the resource creation logic. This will call the sub module with all the parameters needed for the particular resource you want to deploy, inside the /module/resourcename folder. When there are a lot of resouces of the same type, they can be grouped into a subdirectory (for instance, all network-related resources are under /module/networking) |
| README.MD        | Short description of the features the module is achieving, the input and output variables.                                                                                                                                                                                                                                                                                   |
| UPGRADE.MD       | Contains upgrade instructions if anyfor module update inside a landing zone.                                                                                                                                                                                                                                                                                                 |

### Sub modules file structure

For each sub module directory, you should have the following files:

| Filename     | Content                                                                                                |
|--------------|--------------------------------------------------------------------------------------------------------|
| main.tf      | Contains the version requirements, for providers, data sources if needed.                              |
| variables.tf | Contains the input variables for the whole module.                                                     |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aztfmod/terraform-azurerm-caf](https://github.com/aztfmod/terraform-azurerm-caf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->

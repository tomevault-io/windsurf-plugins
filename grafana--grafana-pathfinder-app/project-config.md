---
trigger: always_on
description: This document describes the requirements and design constraints of interactive guides found in this repository.
---


# Interactive Elements Requirements System

This document describes the requirements and objectives system for interactive elements in the Grafana Documentation Plugin.

For full reference on interactive guide authoring (JSON format, action types, selectors, guided interactions), see `docs/developer/interactive-examples/`.

## Overview

Interactive elements in documentation content have a system of requirements that must be satisfied before they can be executed, and a potential system of objectives or goals, that the
guides are trying to accomplish.  This document lays all of that out.

1. **Docs Plugin Requirements**: Built-in system rules that govern interactive behavior
2. **Guide-Specific Requirements**: Declared in the `data-requirements` attribute by content authors
3. **Guide-Specific Objectives**: Declared in `data-objectives` attribute by content authors.

## Docs Plugin Requirements

Docs plugin requirements are automatically enforced by the system and control interactive behavior.
The objective of these requirements are as follows:

1. Make it hard for the user to take confusing or nonsense actions (doing steps out of order)
2. Maximize the chances that the workflow goes smoothly as the user expects
3. Maximize safety: the docs-plugin should not allow you to take a step which can't succeed, as
this creates negative surprise.

## Guide-Specific Requirements

Guide-specific requirements are declared in the `data-requirements` attribute of interactive elements. Multiple requirements are comma-separated. It is expected that this list will grow 
over time as we have new kinds of interactive guides. For example, one guide might require
that you have Alloy set up sending data to Grafana Cloud before you can proceed with learning
how the Kubernetes product works; for that guide, we'd have to extend data-requirements so that
the docs-plugin could check that condition was true.

### Currently Supported Requirements

#### `exists-reftarget`
- **Purpose**: Ensures the target element exists in the DOM before the interactive action can be executed
- **Usage**: Most common requirement for interactive elements
- **Example**: `data-requirements="exists-reftarget"`

#### `has-datasources`
- **Purpose**: Ensures that at least one data source exists in Grafana
- **Usage**: For interactive elements that require any data source to be configured
- **Example**: `data-requirements="has-datasources"`

#### `has-datasource:name`
- **Purpose**: Ensures that a data source exists with the specified name or type
- **Usage**: The value is matched against both the data source name AND type (case-insensitive). First match wins.
- **Example**: `data-requirements="has-datasource:prometheus"` matches a data source named "prometheus" OR of type "prometheus"

#### `has-plugin:plugin-id`
- **Purpose**: Ensures that a given plugin is installed and enabled
- **Usage**: plugin-id should match Grafana's plugin ID concept, e.g. `volkovlabs-rss-datasource`
- **Example**: `data-requirements="has-plugin:volkovlabs-rss-datasource"`

#### `has-dashboard-named:name`
- **Purpose**: Ensures that a dashboard exists with the specified exact title
- **Usage**: dashboard-name should match the exact dashboard title in Grafana (case-insensitive)
- **Example**: `data-requirements="has-dashboard-named:Foobar"`

#### `has-permission:action`
- **Purpose**: Ensures the current user has the specified Grafana permission
- **Usage**: Uses Grafana's permission system to check user access
- **Example**: `data-requirements="has-permission:dashboards:write"`

#### `has-role:role`
- **Purpose**: Ensures the current user has the specified organizational role
- **Usage**: Supports roles: admin, editor, viewer, or grafana-admin
- **Example**: `data-requirements="has-role:admin"`

#### `is-admin`
- **Purpose**: Ensures the current user has Grafana admin privileges
- **Usage**: For interactive elements that require admin access to function properly
- **Example**: `data-requirements="is-admin"`

#### `navmenu-open`
- **Purpose**: Ensures the Grafana navigation menu is open/visible
- **Usage**: For interactive elements that need to interact with navigation menu items
- **Example**: `data-requirements="navmenu-open"`

#### `on-page:path`
- **Purpose**: Ensures the user is currently on a specific page/URL path
- **Usage**: Supports both partial and exact path matching
- **Example**: `data-requirements="on-page:/dashboard"`

#### `has-feature:toggle`
- **Purpose**: Ensures that a specific Grafana feature toggle is enabled
- **Usage**: Check if experimental or optional features are available
- **Example**: `data-requirements="has-feature:alerting"`

#### `in-environment:env`
- **Purpose**: Ensures the guide runs in a specific environment
- **Usage**: Useful for dev vs prod specific guide
- **Example**: `data-requirements="in-environment:development"`

#### `min-version:x.y.z`
- **Purpose**: Ensures Grafana version meets minimum requirements
- **Usage**: Uses semantic version comparison (major.minor.patch)
- **Example**: `data-requirements="min-version:9.0.0"`

#### `section-completed:sectionId`
- **Purpose**: Creates dependencies between tutorial sections, ensuring prerequisite sections are completed first

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [grafana/grafana-pathfinder-app](https://github.com/grafana/grafana-pathfinder-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
